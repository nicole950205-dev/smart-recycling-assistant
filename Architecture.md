# Architecture

```text
               SMART RECYCLING ASSISTANT
 ┌────────────────────────────────────────────┐
 │                AMB82-Mini                  │
 │                                            │
 │ Camera ──> Image Capture ──> Image Request │
 │ IR Sensor ──> Object / Presence Detection  │
 │                                            │
 │          Recycling Agent API               │
 │         HTTP / JSON over Wi-Fi             │
 │                    │                       │
 │       ┌────────────┼────────────┐          │
 │       ▼            ▼            ▼          │
 │    Status        LED /        Servo       │
 │    Control       Audio       (Optional)    │
 │                 Feedback                   │
 └────────────────────┬───────────────────────┘
                      │ Wi-Fi
                      │
              Android phone/tablet
 ┌────────────────────▼───────────────────────┐
 │          Google AI Edge Gallery            │
 │                                            │
 │          Gemma 4 E2B / AI Model            │
 │                 │                          │
 │          Recycling Agent Logic             │
 │                 │                          │
 │        Classification / Validation         │
 │                 │                          │
 │       ┌─────────┼─────────┐                │
 │       ▼         ▼         ▼                │
 │   classify()  explain()  confirm()         │
 │   get_status() display()  sort()           │
 └────────────────────────────────────────────┘
```
