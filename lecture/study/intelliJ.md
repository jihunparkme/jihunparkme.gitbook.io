# IntelliJ

## Plugins.

- `CodeMetrics`
- `GitHub Copilot`
- `Translation`
  - Translate : Ctrl + Command + U
  - Show Translation Dialog : Ctrl + Command + I
  - Translate and Replace : Ctrl + Command + O

## Setting.

**File and Code Templates**
- Preferences -> Editor -> File and Code Templates
  - New Method Body: `throw new UnsupportedOperationException("${SIMPLE_CLASS_NAME}#${METHOD_NAME} unsupported yet !!");`
  - Implemented Method Body: `throw new UnsupportedOperationException("${SIMPLE_CLASS_NAME}#${METHOD_NAME} not implemented yet !!");`

**Postfix Completion**
- Preferences -> Editor -> General -> Postfix Completion
  - aste: `org.junit.jupiter.api.Assertions.assertEquals("$END$", $EXPR$);`
  - astt: `org.junit.jupiter.api.Assertions.assertTrue($EXPR$);`
  - use static import possible check
  
**intellij introduce variable final**
- Preference -> Editor -> Code Style -> Java -> Code Generation -> Final Modifier
  - Make generated local variables final
  - Make generated parameters final

**Keymap**
- Preferences -> Keymap
  - Select File in Project View -> Add Keyboard Shortcut

**Live Templates**
- Preferences -> Editor -> Live Templates -> Applicable contexts
  - asst: `org.junit.jupiter.api.Assertions.assertEquals($END$, $EXPR$);` 
  - asth: `org.assertj.core.api.Assertions.assertThatThrownBy(() -> { $END$ }).isInstanceOf($EXPR$.class).hasMessageContaining("");` 
  - d: `@org.junit.jupiter.api.DisplayName("$EXPR$")`
  - td: `@org.junit.jupiter.api.Test void $EXPR$() { org.assertj.core.api.Assertions.assertThat($END$).isEqualTo(); }`

**Inspections**
- Preferences -> Editor -> Inspections
  - Local variable or parameter can be final

## Refactor

**extract variable final**
- option + command + v
- option + shift + O
  - Declare final
  - Declare var

**extract method final**
- option + command + m
- option + shift + O
  - make static

**code vision usages**
- Preferences -> Editor -> Inlay Hints -> Java -> Code vision -> Show hints for
  - Usages
  - inheritors