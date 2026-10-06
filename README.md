# AgentFlow Studio

A standalone, single-file multi-agent workflow app. The six-stage workflow plans a question, gathers evidence, compares options, checks claims, writes a brief, and reviews it.

## Run

Open `index.html` in a modern browser. Demo mode works without API keys. For live mode, enter a Gemini API key and/or a Tavily API key in the app, then select **Live with Gemini**. Gemini is preferred when configured; Tavily supplies search-backed fallback responses.

API keys are stored in the browser's local storage for this file and are not included in this repository. They are sent directly from the browser to the selected provider. Do not commit keys or share a copy of the app containing credentials.
