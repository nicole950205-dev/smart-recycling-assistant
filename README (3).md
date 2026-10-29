# AI-smart-recycling-assistant
![](assets/smart_recycling_assistant.png?raw=true)

---
## System HW & SW specifications

### System Block Diagram
![](assets/AI-smart-recycling_block_diagram.png?raw=true)

### LLM + Agent
`gemma4:e2b` based on [Google-AI-Edge Gallery](https://github.com/google-ai-edge/gallery) <br>

### Hardware :
1. EVB : AMB82-Mini (built-in camera)
2. Sensor : IR sensor
3. Feedback : LED + audio (speaker)
4. Optional : Servo + sorting bins

---
## Prompts:
`GPT6.1`<br>
```
Google Edge AI Gallery App support MCP and Skills, what would be recommended code to run on AMB82-mini to host on the smart recycling assistant
```

---
## [Architecture.md](Architecture.md)
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
### System Architecture Diagram
![](assets/AI-smart-recycling_architecture_diagram.png?raw=true)

---
## ProtoTyping

### Prompt
`GPT6.1`<br>
```
I would like to design a smart recycling assistant with Google Edge AI Gemma4-E2B model using AMB82-mini. AMB82-Mini offers features such as image capture, IR sensor, LED / audio feedback, and optional servo control. The camera on AMB82-mini captures an image of the item, sends it over Wi-Fi to an Android phone running Google AI Edge Gallery, where the Gemma4-E2B model classifies it (plastic bottle, paper, metal can, or unknown) and returns a result for feedback, user confirmation, and optional automatic sorting. Generate a Proposal.md
```

### [Proposal.md](Proposal.md)
