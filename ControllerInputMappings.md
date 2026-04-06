# Controller Input Mappings

This project uses Unity's legacy Input Manager mappings in `ProjectSettings/InputManager.asset`.

## Configured actions

| Control | Unity mapping name | Type | Binding |
| --- | --- | --- | --- |
| Left Stick X | `Move Horizontal` | Axis | Joystick axis 1 |
| Left Stick Y | `Move Vertical` | Axis | Joystick axis 2 |
| Right Stick X | `Look Horizontal` | Axis | Joystick axis 4 |
| Right Stick Y | `Look Vertical` | Axis | Joystick axis 5 |
| Left Trigger | `Sprint` | Axis | Joystick axis 9 |
| A | `Jump` | Button | `joystick button 0` |
| B | `Drop` | Button | `joystick button 1` |
| X | `Interact` | Button | `joystick button 2` |
| Y | `Skip Dialogue` | Button | `joystick button 3` |
| Right Bumper | `Pickup / Throw` | Button | `joystick button 5` |

These bindings target the standard Xbox-style controller layout on Windows, which also matches most XInput-compatible controllers in Unity.

## Script usage

Use the shared helper in `Assets/Scripts/GameInput.cs`:

```csharp
Vector2 move = GameInput.GetMoveVector();
Vector2 look = GameInput.GetLookVector();

if (GameInput.IsSprintHeld()) { }
if (GameInput.JumpPressed()) { }
if (GameInput.DropPressed()) { }
if (GameInput.InteractPressed()) { }
if (GameInput.SkipDialoguePressed()) { }
if (GameInput.PickupThrowPressed()) { }
```
