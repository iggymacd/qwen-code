### Implementation Plan: Agent Abstraction Layer

**Objective:** Refactor the existing agentic logic into a modular, reusable, and extensible framework by creating an Agent Abstraction Layer.

**Estimated Time:** 3-5 developer days

**Key Milestones:**

1.  **Core Interfaces Defined:** All new interfaces (`Agent`, `LanguageModel`, `LLMResponse`) are defined and documented.
2.  **`GeminiModel` Implemented:** The `GeminiModel` class is created and tested.
3.  **`GeminiCliAgent` Implemented:** The `GeminiCliAgent` class is created and tested.
4.  **CLI Integration:** The `gemini.tsx` file is updated to use the new `GeminiCliAgent`.
5.  **Testing and Validation:** All existing and new functionality is fully tested.

---

### **Phase 1: Core Interface Definition**

**Task 1.1: Define the `LanguageModel` and `LLMResponse` Interfaces**

*   **File:** `packages/core/src/agent/language-model.ts` (new file)
*   **Details:**
    *   Create a `LanguageModel` interface with a single method: `generateResponse(history: ConversationTurn[], tools: Tool[]): Promise<LLMResponse>`.
    *   Create an `LLMResponse` interface with two optional properties: `text?: string` and `toolCalls?: ToolCallRequestInfo[]`.

**Task 1.2: Define the `Agent` Interface**

*   **File:** `packages/core/src/agent/agent.ts` (new file)
*   **Details:**
    *   Create an `Agent` interface.
    *   Define the `run(prompt: string, signal: AbortSignal): Promise<AgentResult>` method.
    *   Define the event-based API: `on(event: string, handler: (...args: any[]) => void): this`.
    *   Define the `AgentResult` interface with `finalResponse: string` and `completedToolCalls: CompletedToolCall[]`.

---

### **Phase 2: Concrete `LanguageModel` Implementation**

**Task 2.1: Create the `GeminiModel` Class**

*   **File:** `packages/core/src/agent/gemini-model.ts` (new file)
*   **Details:**
    *   Create a `GeminiModel` class that implements the `LanguageModel` interface.
    *   Move the existing Gemini API call logic from its current location (likely within the `generateContent` stream handler) into the `generateResponse` method of this class.
    *   This class will take the necessary configuration (API key, etc.) in its constructor.

**Task 2.2: Unit Test the `GeminiModel` Class**

*   **File:** `packages/core/src/agent/gemini-model.test.ts` (new file)
*   **Details:**
    *   Write unit tests for the `GeminiModel` class.
    *   Mock the Gemini API client to test the `generateResponse` method without making actual API calls.
    *   Ensure that the method correctly formats the request and parses the response.

---

### **Phase 3: Concrete `Agent` Implementation**

**Task 3.1: Create the `GeminiCliAgent` Class**

*   **File:** `packages/core/src/agent/gemini-cli-agent.ts` (new file)
*   **Details:**
    *   Create a `GeminiCliAgent` class that implements the `Agent` interface.
    *   The constructor will accept a `LanguageModel` instance and a `ToolRegistry` instance.
    *   The `run` method will contain the main agent loop:
        1.  Call the `LanguageModel` to get a response.
        2.  If the response contains tool calls, instantiate and use the `CoreToolScheduler` to execute them.
        3.  If the response contains text, emit the `final-response` event.
        4.  Repeat until a final text response is received.
    *   The `CoreToolScheduler`'s event handlers will be used to emit events from the `GeminiCliAgent`.

**Task 3.2: Unit Test the `GeminiCliAgent` Class**

*   **File:** `packages/core/src/agent/gemini-cli-agent.test.ts` (new file)
*   **Details:**
    *   Write unit tests for the `GeminiCliAgent`.
    *   Use a mock `LanguageModel` to control the responses and test the agent's logic.
    *   Test the agent's ability to handle text responses, tool calls, and errors.
    *   Verify that the correct events are emitted.

---

### **Phase 4: Integration and Refactoring**

**Task 4.1: Refactor `gemini.tsx` to Use the `GeminiCliAgent`**

*   **File:** `packages/cli/src/gemini.tsx`
*   **Details:**
    *   Remove the direct dependency on `CoreToolScheduler`.
    *   Instantiate the `GeminiModel` and `GeminiCliAgent`.
    *   Use the `agent.on(...)` methods to subscribe to events and update the UI.
    *   Call `agent.run(...)` to process user input.

**Task 4.2: End-to-End Testing**

*   **Details:**
    *   Manually test the CLI to ensure that all existing functionality is working as expected.
    *   Test the interactive and non-interactive modes.
    *   Test tool calls that require confirmation.
    *   Test error handling.

---

### **Phase 5: Documentation and Cleanup**

**Task 5.1: Update Documentation**

*   **File:** `docs/architecture.md` and `docs/agent-abstraction-prd.md`
*   **Details:**
    *   Update the architecture documentation to reflect the new Agent Abstraction Layer.
    *   Add a new document explaining the agent framework and how to extend it.
    *   Mark the PRD as "Implemented".

**Task 5.2: Code Cleanup**

*   **Details:**
    *   Remove any old, unused code that was replaced by the new framework.
    *   Ensure that the new code adheres to the project's coding standards.