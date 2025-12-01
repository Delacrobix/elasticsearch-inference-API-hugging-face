# Content Moderation Pipeline Flow

```mermaid
flowchart TD
    Start([Document Ingested]) --> P1
    
    P1["Processor 1: Script<br/>Build evaluation_prompt<br/><i>system_prompt + title + content</i>"]
    P1 --> |evaluation_prompt| P2
    
    P2["Processor 2: Inference<br/>Call Hugging Face Model<br/><i>llama3.2 via Ollama</i>"]
    P2 --> |evaluation_result<br/>JSON string with markdown| P3
    
    P3["Processor 3: Script<br/>Clean Markdown Wrappers<br/><i>Remove ```json blocks</i>"]
    P3 --> |cleaned JSON string| P4
    
    P4["Processor 4: JSON Parser<br/>Parse String to Object<br/><i>evaluation_parsed</i>"]
    P4 --> |evaluation_parsed object| P5
    
    P5["Processor 5: Script<br/>Extract Fields<br/><i>safe_content & moderation_analysis</i>"]
    P5 --> |Final fields added| P6
    
    P6["Processor 6: Remove<br/>Clean Temporary Fields<br/><i>Remove evaluation_*</i>"]
    P6 --> End
    
    End([Document Indexed with<br/>safe_content & moderation_analysis])
    
    style P1 fill:#e1f5ff,stroke:#0288d1,stroke-width:2px
    style P2 fill:#fff3e0,stroke:#f57c00,stroke-width:2px
    style P3 fill:#e1f5ff,stroke:#0288d1,stroke-width:2px
    style P4 fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px
    style P5 fill:#e1f5ff,stroke:#0288d1,stroke-width:2px
    style P6 fill:#ffebee,stroke:#c62828,stroke-width:2px
    style Start fill:#e8f5e9,stroke:#388e3c,stroke-width:3px
    style End fill:#e8f5e9,stroke:#388e3c,stroke-width:3px
```

## Pipeline Stages Explained

### Stage 1: Build Prompt (Script)
- **Input**: Document fields (`title`, `content`)
- **Action**: Concatenates system prompt with article data
- **Output**: `evaluation_prompt` (text string)
- **Purpose**: Prepare the complete prompt for the LLM

### Stage 2: Model Inference
- **Input**: `evaluation_prompt`
- **Action**: Calls Hugging Face endpoint (Ollama/llama3.2)
- **Output**: `evaluation_result` (JSON wrapped in markdown)
- **Purpose**: Get AI-powered content moderation decision

### Stage 3: Clean Markdown (Script)
- **Input**: `evaluation_result`
- **Action**: Removes markdown code block wrappers (` ```json `)
- **Output**: Clean JSON string
- **Purpose**: Prepare for JSON parsing

### Stage 4: JSON Parser
- **Input**: Cleaned JSON string
- **Action**: Converts string to object
- **Output**: `evaluation_parsed` (object)
- **Purpose**: Make data accessible for field extraction

### Stage 5: Extract Fields (Script)
- **Input**: `evaluation_parsed`
- **Action**: Extracts `safe_content` and `moderation_analysis`
- **Output**: Document with final fields
- **Purpose**: Store moderation results in final schema

### Stage 6: Cleanup (Remove)
- **Input**: Document with all fields
- **Action**: Removes temporary fields (`evaluation_*`)
- **Output**: Clean final document
- **Purpose**: Keep only necessary fields in index
