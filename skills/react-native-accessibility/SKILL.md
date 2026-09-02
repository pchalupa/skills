---
name: react-native-accessibility
description: React Native accessibility props (VoiceOver and TalkBack) plus unit testing practices that query by role and label instead of testID or UNSAFE_* APIs. Use when writing, reviewing, or testing RN UI.
---

# React Native Accessibility

Use this skill when writing, reviewing, or refactoring React Native UI, and when writing tests for RN components. It covers every accessibility prop VoiceOver and TalkBack actually care about, with concrete code patterns, plus how to write tests that query the same tree a screen reader does.

Two rules drive everything below:

1. **Every interactive element needs a role and a label.** If a screen reader can't describe it, neither can an AI agent, and neither can your test runner.
2. **Tests query by role and label, not by `testID` or component internals.** If the test can find the button, a screen reader user can too. If it can't, that's a real bug, not a testing bug.

The core props (`accessibilityRole`, `accessibilityLabel`, `accessibilityState`) work the same on iOS and Android. Platform-specific props and quirks are called out inline.

---

## 1. Roles, Labels, and Hints

Every interactive element needs three things: what it is (`accessibilityRole`), what it's called (`accessibilityLabel`), and optionally what it does (`accessibilityHint`). VoiceOver announces: "label, hint, role". TalkBack announces the same.

```tsx
// bad: plain View acting as a button
<View onPress={() => open(ability)}>
  <Image source={{ uri: ability.icon }} />
  <Text>{ability.name}</Text>
</View>

// good: Pressable with explicit role and label
<Pressable
  onPress={() => open(ability)}
  accessibilityRole="button"
  accessibilityLabel={`${ability.name}. Double tap for details.`}
>
  <Image source={{ uri: ability.icon }} />
  <Text>{ability.name}</Text>
</Pressable>
```

- Prefer `Pressable` over `TouchableOpacity` in modern RN.
- Hints can be disabled by the user on iOS. On Android, TalkBack always reads them. **Never put critical info only in a hint.** If the user needs it, put it in the label.
- Label describes the action, not the visual. "Close ability details", not "X icon".

### Supported `accessibilityRole` values

Form controls: `checkbox`, `radio`, `radiogroup`, `combobox`, `spinbutton`, `switch`, `togglebutton`.
Content: `header`, `link`, `image`, `imagebutton`, `text`, `alert`, `summary`.
Navigation: `menu`, `menubar`, `menuitem`, `tab`, `tablist`, `toolbar`, `grid`.
Interactive: `button`, `keyboardkey`, `adjustable`, `progressbar`, `scrollbar`, `search`, `timer`.
No role: `none`.

### `role` (newer, preferred)

RN also supports the web-aligned `role` prop, which takes precedence over `accessibilityRole` when both are set. It adds values `accessibilityRole` doesn't have: `heading` (instead of `header`), `img`, `list`, `listitem`, `searchbox`, `slider`, `presentation`.

### Images

Meaningful images need descriptive labels. Decorative images should be hidden from the accessibility tree entirely, otherwise the screen reader just says "Image", which is useless.

```tsx
// bad: missing alt text, screen reader just says "Image"
<Image source={{ uri: championSplashUrl }} />

// good: descriptive label
<Image
  source={{ uri: championSplashUrl }}
  accessibilityLabel="Ahri, the Nine-Tailed Fox"
/>

// good: decorative image hidden from both platforms
<Image
  source={{ uri: backgroundUrl }}
  accessibilityElementsHidden={true}              // iOS
  importantForAccessibility="no-hide-descendants" // Android
/>
```

### The uppercase trap

`textTransform: 'uppercase'` and `letterSpacing` on Android both affect the text passed to the accessibility tree. TalkBack reads each letter individually. The fix is an explicit `accessibilityLabel`, which TalkBack uses instead of the visual text.

---

## 2. ARIA aliases

Modern React Native accepts ARIA-style props alongside the `accessibility*` names. They map 1:1.

| ARIA prop         | Maps to                                                                           |
| ----------------- | --------------------------------------------------------------------------------- |
| `aria-label`      | `accessibilityLabel`                                                              |
| `aria-hidden`     | `accessibilityElementsHidden` / `importantForAccessibility="no-hide-descendants"` |
| `aria-disabled`   | `accessibilityState.disabled`                                                     |
| `aria-checked`    | `accessibilityState.checked` (accepts `'mixed'`)                                  |
| `aria-expanded`   | `accessibilityState.expanded`                                                     |
| `aria-selected`   | `accessibilityState.selected`                                                     |
| `aria-busy`       | `accessibilityState.busy`                                                         |
| `aria-live`       | `accessibilityLiveRegion` (Android)                                               |
| `aria-modal`      | `accessibilityViewIsModal` (iOS)                                                  |
| `aria-valuemin`   | `accessibilityValue.min`                                                          |
| `aria-valuemax`   | `accessibilityValue.max`                                                          |
| `aria-valuenow`   | `accessibilityValue.now`                                                          |
| `aria-valuetext`  | `accessibilityValue.text`                                                         |
| `aria-labelledby` | `accessibilityLabelledBy` (Android)                                               |
| `role`            | `accessibilityRole` (takes precedence)                                            |

Pick one style per component and stick with it. Mixing `aria-label` and `accessibilityLabel` on the same element is confusing, and `role` silently wins over `accessibilityRole`.

---

## 3. Accessibility State

Interactive components often have dynamic state: checked, disabled, expanded, selected. If the screen reader doesn't know, the user is flying blind.

```tsx
// bad: TalkBack says "Mage, button" whether selected or not
<Pressable
  onPress={() => toggle(role)}
  accessibilityRole="button"
  accessibilityLabel={role}
>
  <Text>{role}</Text>
</Pressable>

// good: TalkBack says "Mage, selected, button" and announces on change
<Pressable
  onPress={() => toggle(role)}
  accessibilityRole="button"
  accessibilityLabel={role}
  accessibilityState={{ selected }}
>
  <Text>{role}</Text>
</Pressable>

// disabled state on Apply button
<Pressable
  onPress={applyFilter}
  disabled={!hasSelection}
  accessibilityRole="button"
  accessibilityLabel="Apply filter"
  accessibilityState={{ disabled: !hasSelection }}
>
  <Text>Apply filter</Text>
</Pressable>
```

Available states: `selected`, `disabled`, `checked` (can be `'mixed'`), `expanded`, `busy`.

**Don't bake state into the label** (e.g. `"Submit - disabled"`). Let `accessibilityState` handle it. Both screen readers know how to announce these states natively and will auto-announce state changes on tap.

### Adjustable controls

For sliders, steppers, and range inputs, use `accessibilityRole="adjustable"` with `accessibilityValue` and `increment` / `decrement` actions. Without the actions, the control is announce-only.

```tsx
<View
  accessible
  accessibilityRole="adjustable"
  accessibilityLabel="Champion difficulty filter"
  accessibilityValue={{ min: 1, max: 10, now: difficulty, text: `${difficulty} of 10` }}
  accessibilityActions={[
    { name: "increment", label: "Increase difficulty" },
    { name: "decrement", label: "Decrease difficulty" },
  ]}
  onAccessibilityAction={(event) => {
    if (event.nativeEvent.actionName === "increment") inc();
    if (event.nativeEvent.actionName === "decrement") dec();
  }}
>
  <DifficultyPips value={difficulty} />
  {/* hide the visual +/- buttons, adjustable gestures replace them */}
  <View importantForAccessibility="no">
    <Controls value={difficulty} onDec={dec} onInc={inc} />
  </View>
</View>
```

Without this, the screen reader lands on three separate controls: "button", "5 / 10", "button". With it: "Champion difficulty filter, 5 of 10, adjustable". Users swipe up/down to change the value.

---

## 4. Element Grouping

Grouping related children is one of the quickest wins for screen reader users. A match card with champion name, role, result, KDA, CS, and duration is six focus stops without grouping, one with.

```tsx
// bad: 6 swipes to read one card, each field read in isolation
<View style={styles.card}>
  <Text>{match.champion}</Text>
  <Text>{match.role}</Text>
  <Text>{match.result}</Text>
  <Text>{`${match.k}/${match.d}/${match.a}`}</Text>
  <Text>{`${match.cs} CS`}</Text>
  <Text>{match.duration}</Text>
</View>

// good: 1 swipe reads the full card
// TalkBack: "Ahri, Mid, Victory, 8/3/12 KDA, 187 CS, 34:21"
<View
  accessible
  accessibilityLabel={`${match.champion}, ${match.role}, ${match.result}, ${match.k}/${match.d}/${match.a} KDA, ${match.cs} CS, ${match.duration}`}
  style={styles.card}
>
  <Text>{match.champion}</Text>
  <Text>{match.role}</Text>
  <Text>{match.result}</Text>
  <Text>{`${match.k}/${match.d}/${match.a}`}</Text>
  <Text>{`${match.cs} CS`}</Text>
  <Text>{match.duration}</Text>
</View>
```

Without grouping, a list of 20 match cards is 120+ swipes. With it, 20.

### The nested interactive trap

`accessible={true}` on a parent makes the whole subtree an atomic element. On iOS this maps to `isAccessibilityElement = true`, which explicitly prohibits nested accessibility elements. Android behaves similarly. **Any interactive children become unreachable to screen readers.**

```tsx
// bad: "View abilities" button is unreachable inside an accessible container
<View
  accessible
  accessibilityLabel="Ahri, the Nine-Tailed Fox. Tags: Mage, Assassin"
>
  <Text>Ahri</Text>
  <Pressable onPress={() => openAbilities('Ahri')}>
    <Text>View abilities</Text>
  </Pressable>
</View>

// good: use a custom action for the secondary behaviour
<View
  accessible
  accessibilityLabel="Ahri, the Nine-Tailed Fox. Tags: Mage, Assassin"
  accessibilityActions={[{ name: 'viewAbilities', label: 'View abilities' }]}
  onAccessibilityAction={(event) => {
    if (event.nativeEvent.actionName === 'viewAbilities') {
      openAbilities('Ahri');
    }
  }}
>
  <Text>Ahri</Text>
  <Text importantForAccessibility="no-hide-descendants">View abilities</Text>
</View>
```

---

## 5. Custom Actions

Gestures (swipe to delete, long-press for options, drag to reorder) are invisible to screen readers. Custom actions expose them.

### Standard action names

| Action      | iOS | Android | Trigger                                |
| ----------- | --- | ------- | -------------------------------------- |
| `activate`  | yes | yes     | Double-tap with screen reader on       |
| `increment` | yes | yes     | Swipe up / volume up                   |
| `decrement` | yes | yes     | Swipe down / volume down               |
| `magicTap`  | yes | no      | Two-finger double-tap (primary action) |
| `escape`    | yes | no      | Two-finger Z gesture (back / dismiss)  |
| `longpress` | no  | yes     | Double-tap and hold                    |
| `expand`    | no  | yes     | Expand a collapsible element           |
| `collapse`  | no  | yes     | Collapse a collapsible element         |

### Custom actions

```tsx
<View
  accessible
  accessibilityActions={[
    { name: "viewAbilities", label: "View abilities" },
    { name: "addToTeam", label: "Add to team" },
  ]}
  onAccessibilityAction={(event) => {
    switch (event.nativeEvent.actionName) {
      case "viewAbilities":
        openAbilitySheet(champion);
        break;
      case "addToTeam":
        addToTeam(champion);
        break;
    }
  }}
  accessibilityLabel={`${champion.name}, ${champion.title}`}
>
  <Image source={{ uri: champion.icon }} />
  <Text>{champion.name}</Text>
  <Text>{champion.title}</Text>
</View>
```

VoiceOver users swipe up/down to cycle through actions. TalkBack users get the same via the local context menu.

### iOS-only gesture handlers

These fire on specific VoiceOver gestures and don't need to be declared in `accessibilityActions`.

- `onMagicTap` - two-finger double-tap anywhere. Single "primary action" (play/pause, end call, send).
- `onAccessibilityEscape` - two-finger Z scrub. Close modals, sheets, menus. Same as a back button.
- `onAccessibilityTap` - double-tap while the element has VoiceOver focus. Rarely needed, overrides default activation.

```tsx
<View onAccessibilityEscape={close}>
  <AbilityDetailSheet ability={selected} />
</View>
```

---

## 6. Live Regions

If a piece of UI updates dynamically (status message, loading indicator, form validation error), the screen reader needs to know. Live regions announce text content changes automatically. No manual API calls.

```tsx
// bad: status changes on screen but TalkBack stays silent
<Text accessibilityLiveRegion="none">{statusText}</Text>

// good: TalkBack announces "Sending invite..." then "Invite sent to Faker."
<Text accessibilityLiveRegion="polite">{statusText}</Text>
// or the ARIA form
<Text aria-live="polite">{statusText}</Text>
```

Values:

- `"none"` - no announcement (default)
- `"polite"` - announces when user is idle (most cases)
- `"assertive"` - interrupts immediately (errors, urgent alerts only)

**Android gotcha:** put `accessibilityLiveRegion` on the `Text` element, not a wrapper `View`. Announcements fire on content change, not on mount. Keep the element always rendered and change its text.

```tsx
// bad: TalkBack often misses this, element mounts/unmounts
{
  status && (
    <View accessibilityLiveRegion="polite">
      <Text>{status}</Text>
    </View>
  );
}

// good: element stays in the tree, content changes
<Text accessibilityLiveRegion="polite">{status}</Text>;
```

---

## 7. Imperative Announcements

For side-effects that have no visible text change (async completions, background sync, toasts the user might miss), use `AccessibilityInfo.announceForAccessibility`. Works on both platforms.

```tsx
import { AccessibilityInfo } from "react-native";

const sendInvite = async (player) => {
  setStatus("sending");
  try {
    await inviteToGame(player.id);
    setStatus("sent");
    AccessibilityInfo.announceForAccessibility(`Invite sent to ${player.name}`);
  } catch {
    setStatus("busy");
    AccessibilityInfo.announceForAccessibility(`${player.name} is currently in a game`);
  }
};

const loadChampions = () => {
  setLoading(true);
  AccessibilityInfo.announceForAccessibility("Loading champions");
  fetchChampions().finally(() => {
    setLoading(false);
    AccessibilityInfo.announceForAccessibility("Champions loaded");
  });
};
```

Use for:

- Async action results (invites sent, requests failed)
- Data loading completion
- Toast notifications the user might otherwise miss
- Background sync completing

**When to prefer live regions instead:** any case where text in the UI is changing in place. Live regions tie the announcement to the DOM change, which is more reliable on Android than firing imperatively.

---

## 8. Focus Management

After a state transition (modal opening, sheet dismissing, screen navigating), screen reader focus can land anywhere. Move it deliberately, then return it to the trigger on close.

```tsx
import { useRef } from "react";
import { AccessibilityInfo, findNodeHandle, View, Text } from "react-native";

const abilityRefs = useRef<Record<string, React.RefObject<View>>>({});
const modalTitleRef = useRef<Text>(null);
const lastOpenedKey = useRef<string | null>(null);

const open = (ability) => {
  lastOpenedKey.current = ability.key;
  setSelected(ability);

  // move focus into the sheet after it mounts
  setTimeout(() => {
    const node = findNodeHandle(modalTitleRef.current);
    if (node) AccessibilityInfo.setAccessibilityFocus(node);
  }, 100);
};

const close = () => {
  const key = lastOpenedKey.current;
  setSelected(null);

  // return focus to the button that opened the sheet
  setTimeout(() => {
    if (key) {
      const ref = abilityRefs.current[key];
      const node = findNodeHandle(ref?.current ?? null);
      if (node) AccessibilityInfo.setAccessibilityFocus(node);
    }
  }, 100);
};
```

The `setTimeout` matters. Focus moves after mount, otherwise the target isn't in the native view tree yet.

### Custom focus order

If visual layout doesn't match logical reading order, use `experimental_accessibilityOrder` with `nativeID`s on children. Use sparingly - wrong order breaks things faster than it fixes them.

```tsx
<View experimental_accessibilityOrder={["title", "summary", "cta"]}>
  <View accessible nativeID="cta">
    <Text>Get started</Text>
  </View>
  <Text accessible nativeID="title">
    Welcome
  </Text>
  <Text accessible nativeID="summary">
    A short description.
  </Text>
</View>
```

### Modal containment

Moving focus into a modal isn't enough. Background content can still be swiped to, which is disorienting. Both sides need handling:

- **iOS:** `accessibilityViewIsModal={true}` on the modal view, VoiceOver ignores everything outside.
- **Android:** `importantForAccessibility="no-hide-descendants"` on the background content hides it from TalkBack.

```tsx
// the sheet
<View style={styles.sheet} accessibilityViewIsModal>
  <Text
    ref={modalTitleRef}
    accessibilityRole="header"
    style={styles.sheetTitle}
  >
    {selected.name}
  </Text>
  <Text>{selected.description}</Text>

  {/* group table-like rows so they read in one swipe */}
  {/* without this, "Cooldown", "7s", "Cost", "65 Mana" are 4 focus stops */}
  <View
    accessible
    accessibilityLabel={`Cooldown, ${selected.cooldown}. Cost, ${selected.cost}`}
  >
    <View><Text>Cooldown</Text><Text>{selected.cooldown}</Text></View>
    <View><Text>Cost</Text><Text>{selected.cost}</Text></View>
  </View>

  <Pressable
    onPress={close}
    accessibilityRole="button"
    accessibilityLabel="Close ability details"
  >
    <Text>Close</Text>
  </Pressable>
</View>

// the background content (Android needs this explicitly)
<View importantForAccessibility={selected ? 'no-hide-descendants' : 'auto'}>
  {/* ability buttons behind the sheet */}
</View>
```

Focus management (move on open, return on close) plus containment are both required. One without the other leaves gaps.

---

## 9. Form label association (Android)

For `TextInput`s labelled by a visible `Text`, link them via `nativeID` + `accessibilityLabelledBy`. TalkBack reads the label text when the input is focused.

```tsx
<View>
  <Text nativeID="emailLabel">Email address</Text>
  <TextInput accessibilityLabelledBy="emailLabel" />
</View>
```

iOS doesn't need this. A visible `<Text>` immediately before the `<TextInput>` is enough for VoiceOver. For cross-platform safety, also set `accessibilityLabel` on the input.

---

## 10. Adaptive UI

Sometimes the right call is to detect a screen reader and adapt. Complex gesture interactions (swipe carousels, drag-and-drop) can swap to simpler button-based alternatives. **Don't fork the whole app** - the goal is equivalent functionality, not a different experience.

```tsx
import { useEffect, useState } from "react";
import { AccessibilityInfo } from "react-native";

export const useScreenReader = () => {
  const [isActive, setIsActive] = useState(false);

  useEffect(() => {
    AccessibilityInfo.isScreenReaderEnabled().then(setIsActive);
    const sub = AccessibilityInfo.addEventListener("screenReaderChanged", setIsActive);
    return () => sub.remove();
  }, []);

  return isActive;
};

// usage
const ChampionCard = ({ champion }) => {
  const isScreenReaderActive = useScreenReader();

  return isScreenReaderActive ? (
    <Pressable
      onPress={() => openDetails(champion)}
      accessibilityRole="button"
      accessibilityLabel={`${champion.name}, ${champion.title}. Double tap for details.`}
    >
      <Text>{champion.name}</Text>
    </Pressable>
  ) : (
    <SwipeableRow onSwipe={() => openDetails(champion)}>
      <Text>{champion.name}</Text>
    </SwipeableRow>
  );
};
```

`AccessibilityInfo.isScreenReaderEnabled` covers both VoiceOver and TalkBack. Other useful checks with matching event listeners:

- `isReduceMotionEnabled()` - user wants less animation
- `isBoldTextEnabled()` (iOS)
- `isGrayscaleEnabled()` (iOS)
- `isInvertColorsEnabled()` (iOS)
- `isReduceTransparencyEnabled()` (iOS)

---

## 11. Touch Targets

Minimums: 44x44pt on iOS (Human Interface Guidelines), 48x48dp on Android (Material Design). A 24px icon in a tight grid almost always falls short.

`hitSlop` expands the tappable area without changing the visual layout:

```tsx
<Pressable
  onPress={close}
  accessibilityRole="button"
  accessibilityLabel="Close ability details"
  hitSlop={{ top: 10, bottom: 10, left: 10, right: 10 }}
>
  <CloseIcon size={24} />
</Pressable>
```

Add to your component review checklist: anything with an icon button under 44pt needs a `hitSlop`. Close buttons, share icons, overflow menus. They're almost always too small.

---

## 12. Font Scaling

Users with low vision scale their system font, sometimes dramatically. RN's `Text` respects this by default: `allowFontScaling={true}` on every `Text`. Don't disable without a reason.

```tsx
// bad: disables scaling, breaks it for users who need it
<Text allowFontScaling={false}>Orb of Deception</Text>

// bad: fixed height clips the description at large font sizes
<View style={{ height: 48 }}>
  <Text>{ability.description}</Text>
</View>

// good: minimum height lets the container grow
<View style={{ minHeight: 48 }}>
  <Text>{ability.description}</Text>
</View>
```

The usual culprit for layout breakage at large font sizes is a fixed `height` on a container. Swap for `minHeight` and the container expands with the text.

**Test at the largest system font size before shipping.** On Android: `adb shell settings put system font_scale 2.0`. On iOS: Settings > Accessibility > Display & Text Size > Larger Text, or Xcode's Environment Overrides to toggle Dynamic Type live.

---

## 13. Other iOS props worth knowing

- `accessibilityLanguage` - BCP 47 locale code (`"fr-FR"`, `"ja-JP"`). Tells VoiceOver to pronounce this element with a different voice. Critical for multilingual content.
- `accessibilityIgnoresInvertColors` - set `true` on photos and media so Smart Invert doesn't destroy the image.
- `accessibilityShowsLargeContentViewer` + `accessibilityLargeContentTitle` - shows a system-level zoom tooltip on long-press, useful for tiny icons.
- `accessibilityElementsHidden` - hide element and descendants from VoiceOver.

## 13a. Other Android props worth knowing

- `importantForAccessibility` - `"auto" | "yes" | "no" | "no-hide-descendants"`. The `no-hide-descendants` value is the Android equivalent of `accessibilityElementsHidden`.
- `UIManager.sendAccessibilityEvent(node, type)` - programmatically fire `typeViewFocused`, `typeViewClicked`, or `typeWindowStateChanged`. Rarely needed.

---

## 14. Unit Testing: query like a screen reader

The selector you use in a test should be what a screen reader user hears. Tying tests to accessibility props gets you three things at once: resilient tests, accessible components, and an app that works for assistive tech.

### Use `@testing-library/react-native`'s role-based queries

```tsx
import { render, screen, fireEvent } from "@testing-library/react-native";

test("ability button has accessible role and label", () => {
  render(<AbilityButton ability={orbOfDeception} />);

  const button = screen.getByRole("button", {
    name: "Orb of Deception. Double tap for details.",
  });
  expect(button).toBeOnTheScreen();
});

test("role chip announces selected state", () => {
  render(<RoleChip role="Mage" selected={true} />);

  // inline state filter (RTL v12+)
  expect(screen.getByRole("button", { name: "Mage", selected: true })).toBeOnTheScreen();
});

test("disabled apply button", () => {
  render(<Filter hasSelection={false} />);

  expect(screen.getByRole("button", { name: "Apply filter", disabled: true })).toBeOnTheScreen();
});
```

If `getByRole` can't find it, the component is missing `accessibilityRole`, which means a real screen reader user can't find it either. Fix the component, not the test.

### Avoid `testID` as a primary selector

`testID` has its place (E2E hooks into opaque third-party components, or disambiguating two identical labels), but it shouldn't be the default. A `testID` tells you nothing about whether the element is usable. It passes even when the component is invisible to assistive tech.

```tsx
// bad: testID only passes because we invented an attribute for tests
<Pressable testID="submit-button" onPress={submit}>
  <Text>Submit</Text>
</Pressable>;
screen.getByTestId("submit-button");

// good: selector works because the button is accessible
<Pressable accessibilityRole="button" accessibilityLabel="Submit form" onPress={submit}>
  <Text>Submit</Text>
</Pressable>;
screen.getByRole("button", { name: "Submit form" });
```

Same rule for Playwright web tests: prefer `getByRole` and `getByLabel` over `data-testid`. A selector tied to an ARIA role is the same thing a screen reader, Playwright, and an AI agent all rely on.

### Never reach for `UNSAFE_` APIs

`UNSAFE_getAllByProps`, `UNSAFE_getByType`, and friends bypass the accessibility tree entirely. They query internal React instances, which means:

- They pass even when the component is completely inaccessible.
- They break the moment you refactor internals (swap `View` for `Pressable`, split a component, bump a library version).
- They tell you nothing about whether a real user can use the thing.

```tsx
// bad: queries internals, coupled to implementation detail
const [submitBtn] = UNSAFE_getAllByProps({ onPress: submit });

// bad: queries by component type, breaks on refactor
const button = UNSAFE_getByType(SubmitButton);

// good: queries the same tree a screen reader uses
const button = screen.getByRole("button", { name: "Submit form" });
```

If the only way to find an element is via `UNSAFE_*`, the element isn't accessible. The test is telling you about a real bug.

### Detox: query by accessibility, not by id

Same principle at the E2E layer. Detox operates on the native view hierarchy and prefers `by.label()` over `by.id()`.

```ts
// good: matches what TalkBack / VoiceOver focuses
await element(by.label("Submit form")).tap();

// fallback: by.id for cases where labels collide
await element(by.id("submit-form-bottom")).tap();
```

### When you genuinely need a `testID`

Only reach for it when:

- You're hooking an E2E framework into a third-party component you can't add `accessibilityLabel` to.
- Two elements share the same role and label, and giving one a different label would change the user experience.

Even then, add accessibility props first and treat `testID` as a last resort.

### Checklist for a component test

- Query by role, label, or text. Not `testID`. Not `UNSAFE_*`.
- Assert on state via inline filters (`selected: true`, `disabled: true`).
- If you added new interactive UI, add a test that finds it via `getByRole`.
- If `getByRole` can't find it, the component is broken, not the test.

---

## 15. Lint rules

Install `eslint-plugin-react-native-a11y` and enable the recommended config. It flags common mistakes at dev time rather than leaving them for manual QA.

```bash
npm install --save-dev eslint-plugin-react-native-a11y
```

```js
// .eslintrc.js
{
  extends: ['plugin:react-native-a11y/all'],
}
```

Top catches:

- `Touchable*` / `Pressable` without `accessibilityRole`.
- Missing `accessibilityLabel` on icon-only buttons.
- Using `accessibilityTraits` (deprecated) instead of `accessibilityRole`.
- `accessibilityRole="button"` without a press handler.

---

## 16. Manual testing

Automated tests catch regressions. Real devices catch experience bugs. Both matter.

- **iOS VoiceOver:** Settings > Accessibility > VoiceOver. Set the triple-click shortcut for fast toggling. Not available in the simulator - test on a real device.
- **Android TalkBack:** Settings > Accessibility > TalkBack. Same gesture model. On an emulator, toggle via `adb` from the terminal (see below).
- **Xcode Accessibility Inspector:** inspect the accessibility tree without VoiceOver. Fast for checking roles and labels.
- **Android Accessibility Scanner:** scans UI for missing labels, small touch targets, low contrast. Available on the Play Store.
- **QA checklists:** add a screen reader pass to PR review for anything touching interactive UI.

### Inspectors vs real devices

Inspectors are good for structure. They can't tell you:

- Whether gesture navigation feels natural
- Whether announcement timing is correct
- Whether focus transitions are disorienting
- Whether the cognitive load of a screen is reasonable

Use inspectors to catch obvious issues quickly. Use real devices with VoiceOver and TalkBack to test the actual experience.

### ADB commands

TalkBack toggle on an emulator:

```bash
# enable
adb shell settings put secure enabled_accessibility_services \
  com.google.android.marvin.talkback/com.google.android.marvin.talkback.TalkBackService \
  && adb shell settings put secure accessibility_enabled 1

# disable
adb shell settings put secure enabled_accessibility_services "" \
  && adb shell settings put secure accessibility_enabled 0
```

Font scale stress testing:

```bash
adb shell settings put system font_scale 1.0   # default
adb shell settings put system font_scale 1.3   # largest in UI
adb shell settings put system font_scale 2.0   # beyond UI maximum
```

Colour vision deficiency simulation:

```bash
adb shell settings put secure accessibility_display_daltonizer_enabled 1
adb shell settings put secure accessibility_display_daltonizer 11  # deuteranomaly (red-green)
adb shell settings put secure accessibility_display_daltonizer 12  # protanomaly (red-green)
adb shell settings put secure accessibility_display_daltonizer 13  # tritanomaly (blue-yellow)
adb shell settings put secure accessibility_display_daltonizer 0   # monochromatic

# disable
adb shell settings put secure accessibility_display_daltonizer_enabled 0
```

Other useful toggles:

```bash
adb shell settings put secure accessibility_display_inversion_enabled 1  # colour inversion
adb shell settings put secure high_text_contrast_enabled 1              # high contrast text
adb shell dumpsys accessibility                                         # dump node tree
```

`dumpsys accessibility` shows exactly what TalkBack sees: content descriptions, roles, states, bounds, available actions. Diff between changes to catch accessibility regressions.

**iOS equivalent:** there's no `adb`. Closest is **Xcode Environment Overrides** (debug bar while the app runs). Toggles Dynamic Type, Bold Text, colour filters, Reduce Motion live. For simulators, `xcrun simctl ui booted appearance dark` toggles dark mode, but font scale and VoiceOver aren't exposed via `simctl`.

---

## 17. Platform Differences Cheatsheet

| Concern              | iOS                                     | Android                                                       |
| -------------------- | --------------------------------------- | ------------------------------------------------------------- |
| Screen reader        | VoiceOver                               | TalkBack                                                      |
| Hints                | User can disable                        | Always read                                                   |
| Hide decorative      | `accessibilityElementsHidden`           | `importantForAccessibility="no-hide-descendants"`             |
| Modal containment    | `accessibilityViewIsModal`              | `importantForAccessibility` on background                     |
| Live regions         | N/A                                     | `accessibilityLiveRegion`, element must stay mounted          |
| Form label link      | Visual proximity + `accessibilityLabel` | `nativeID` + `accessibilityLabelledBy`                        |
| Screen reader in sim | No, real device only                    | Yes via `adb`                                                 |
| Text transform trap  | OK                                      | `textTransform: 'uppercase'` + `letterSpacing` break TalkBack |

---

## 18. Further reading

Current, maintained references only:

- [React Native Accessibility docs](https://reactnative.dev/docs/accessibility)
- [AccessibilityInfo API](https://reactnative.dev/docs/accessibilityinfo)
- [eslint-plugin-react-native-a11y](https://github.com/FormidableLabs/eslint-plugin-react-native-a11y)
- [Google: TalkBack on GitHub](https://github.com/google/talkback)
- [W3C: WCAG 2.1 Quick Reference](https://www.w3.org/WAI/WCAG21/quickref/)
