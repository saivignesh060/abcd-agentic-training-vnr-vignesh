# **Stock Analyst Agent (Telegram \+ n8n \+ AI)**

This workflow transforms a simple Telegram bot into a powerful AI financial analyst. A user can send any stock ticker (e.g., "MSFT"), and the n8n agent will fetch and synthesize technical data from three different API endpoints, then use a SerpApi search tool to find the latest news.

Finally, it combines all of this information into a comprehensive, multi-part analysis and sends it back to the user—all in a single Telegram message.

* **Trigger**: Interactive (via Telegram Trigger)  
* **Data Sources**: Alpha Vantage (x3 endpoints) & SerpApi (Google Search)  
* **AI Core**: Real Time AI Agent node  
* **AI Model**: Google Gemini  
* **Output**: Telegram message

## **Visuals**

### **Workflow Snapshot**

This is the complete n8n workflow, showing the parallel data fetching, merging, and the AI agent architecture.  
![Workflow](https://github.com/saivignesh060/abcd-agentic-training-vnr-vignesh/blob/5ddd0f2c3651d528c2a69bb4248e18ebf03758a3/agentic%20ai%20project/Workflow.png)

### **Final Bot Output**

This is the final, high-quality analysis sent back to the user in Telegram, combining the 5-day trend, momentum (SMA), and real-time news context.  
![Microsoft](https://github.com/saivignesh060/abcd-agentic-training-vnr-vignesh/blob/5ddd0f2c3651d528c2a69bb4248e18ebf03758a3/agentic%20ai%20project/Microsoft.png)
![IBM](https://github.com/saivignesh060/abcd-agentic-training-vnr-vignesh/blob/5ddd0f2c3651d528c2a69bb4248e18ebf03758a3/agentic%20ai%20project/IBM.png)

## **Example AI Output (Text)**

Here is an example of the complete analysis provided by the bot for the "MSFT" ticker:

Here's a comprehensive analysis of Microsoft Corporation:

**Technical Trend:** Over the past five days, Microsoft's stock price has shown a slight downward trend. Starting at $523.61 on October 24th, it rose to $542.07 by October 28th, then dipped to $525.76 on October 30th.

**Momentum:** The latest closing price of $525.76 is above the 20-day Simple Moving Average (SMA) of $521.7205, indicating positive momentum.

**News Context:** A significant factor influencing the recent price movements appears to be the company's latest earnings report. Jim Cramer of Yahoo Finance stated, "Microsoft Reported What I Thought Was a Truly Strong Quarter," which could explain the initial price increase. However, Windows Central reported that "Microsoft obscures OpenAI's $11.5 billion loss last quarter behind corporate and financial jargon in FY26 Q1 earnings," which might be contributing to the more recent dip in price as investors digest this information.

## **High-Level Flow**

User sends ticker (Telegram) → Fetch 3x Technical Data (Alpha Vantage) → Merge Data Streams → AI Agent Analyzes Data & Uses Search Tool (SerpApi) → Send Synthesized Analysis (Telegram)

## **Architecture & Node Responsibilities**

The workflow is built on three parallel data-fetching branches that are combined and fed into a single AI agent.

![Workflow](https://github.com/saivignesh060/abcd-agentic-training-vnr-vignesh/blob/5ddd0f2c3651d528c2a69bb4248e18ebf03758a3/agentic%20ai%20project/Workflow.png)

### **1\. Data Fetching (Parallel)**

* **Telegram Trigger**  
  * Listens for any incoming message from a user.  
  * The message text (e.g., "AAPL") is used as the input for all three data streams.

![Telegram trigger](https://github.com/saivignesh060/abcd-agentic-training-vnr-vignesh/blob/0cb35ddcdb228119362d4f15d8c20c64ed5ade8b/agentic%20ai%20project/Telegram%20trigger.png)

* **Branch 1: 5-Day Trend**  
  * **HTTP Request**: Fetches TIME\_SERIES\_DAILY from Alpha Vantage.  
  * **Edit Fields**: Extracts the 5 most recent closing prices.

![TIMESERIESDAILY](https://github.com/saivignesh060/abcd-agentic-training-vnr-vignesh/blob/0cb35ddcdb228119362d4f15d8c20c64ed5ade8b/agentic%20ai%20project/TIMESERIESDAILY.png)

* **Branch 2: Momentum (SMA)**  
  * **HTTP Request1**: Fetches the 20-day SMA (Simple Moving Average) from Alpha Vantage.  
  * **Edit Fields1**: Extracts the single most recent SMA value.

![SMA](https://github.com/saivignesh060/abcd-agentic-training-vnr-vignesh/blob/0cb35ddcdb228119362d4f15d8c20c64ed5ade8b/agentic%20ai%20project/SMA.png)

* **Branch 3: Company Context**  
  * **HTTP Request2**: Fetches the company OVERVIEW from Alpha Vantage.  
  * **Edit Fields2**: Extracts the full company name.

![OVERVIEW](https://github.com/saivignesh060/abcd-agentic-training-vnr-vignesh/blob/0cb35ddcdb228119362d4f15d8c20c64ed5ade8b/agentic%20ai%20project/OVERVIEW.png)

### **2\. Synthesis & Action (AI Agent)**

* **Merge**  
  * **Mode: Combine (Position)**  
  * This node is crucial. It waits for all three branches to complete and merges their outputs (recent\_prices, latest\_sma, company\_name) into a single item.

![MERGE_NODE](https://github.com/saivignesh060/abcd-agentic-training-vnr-vignesh/blob/0cb35ddcdb228119362d4f15d8c20c64ed5ade8b/agentic%20ai%20project/OVERVIEW.png)

* **Real Time AI Agent**  
  * This is the "brain" of the operation.  
  * **Model Input:** A Google Gemini Chat Model is connected.  
  * **Tool Input:** A SerpApi node is connected as a tool.  
  * **Prompt (User Message):** Feeds the merged data *as text* to the agent.  
  * **System Message (The "Agent's Instructions"):** This is the master prompt that tells the AI *how* to behave.

![AGENT_NODE](https://github.com/saivignesh060/abcd-agentic-training-vnr-vignesh/blob/0cb35ddcdb228119362d4f15d8c20c64ed5ade8b/agentic%20ai%20project/AGENT_NODE.png)
![MODEL_NODE](https://github.com/saivignesh060/abcd-agentic-training-vnr-vignesh/blob/0cb35ddcdb228119362d4f15d8c20c64ed5ade8b/agentic%20ai%20project/MODEL_NODE.png)
![TOOL_NODE](https://github.com/saivignesh060/abcd-agentic-training-vnr-vignesh/blob/de3c820397c494e3fc40174e875902d1e72f4ca3/agentic%20ai%20project/TOOL_NODE.png)

### **3\. Output**

* **Send a text message (Telegram)**  
  * **Chat ID:** {{ $node\["Telegram Trigger"\].json.message.chat.id }} (Replies to the original user).  
  * **Text:** {{ $json.output }} (Takes the final, synthesized output from the AI agent).  
  * **Parse Mode:** None (To avoid Markdown errors from special characters in financial news).

![SEND_A_TEXT_MSG](https://github.com/saivignesh060/abcd-agentic-training-vnr-vignesh/blob/0cb35ddcdb228119362d4f15d8c20c64ed5ade8b/agentic%20ai%20project/SEND_A_TEXT_MSG.png)

## **Key Code Patterns (n8n Expressions)**

This workflow uses JavaScript expressions inside nodes to extract and format data.

1\. Extract 5-Day Prices (Edit Fields)

{{ Object.entries($json\["Time Series (Daily)"\]).slice(0, 5).map(entry \=\> ({ date: entry\[0\], "close": entry\[1\]\["4. close"\] })) }}

This expression converts the large TIME\_SERIES\_DAILY object into a clean 5-item array.

* **Name:** recent\_prices

2\. Extract Latest SMA (Edit Fields1)

{{ Object.values($json\["Technical Analysis: SMA"\])\[0\].SMA }}

This expression digs into the Technical Analysis: SMA object and pulls out the single most recent SMA value.

* **Name:** latest\_sma

3\. Extract Company Name (Edit Fields2)  
{{ $json.Name }}

This expression simply pulls the Name field from the OVERVIEW API response.

* **Name:** company\_name

4\. Format AI Agent Input (Real Time AI Agent \- User Message)

recent\_prices: {{ JSON.stringify($json.recent\_prices) }}  
latest\_sma: {{ $json.latest\_sma }}  
company\_name: {{ $json.company\_name }}

This is the crucial step to prevent the \[object Object\] error. It converts the data into a readable text string for the AI.

5\. AI Agent System Prompt (Real Time AI Agent \- System Message)

This is the master prompt that defines the agent's personality, tasks, and constraints.

“You are a financial analyst agent. Your goal is to provide a comprehensive analysis.

First, you will be given technical data from the previous node. This includes:  
\- \`recent\_prices\`: 5-day price history  
\- \`latest\_sma\`: 20-day SMA  
\- \`company\_name\`: The company's name

Second, you MUST use the \`SerpApi\` tool to find out \*why\* the price has moved. Formulate a good \`search\_query\` (like the company's name \+ "financial news") to get this context.

Finally, synthesize \*all\* information (technicals \+ news) into a single, comprehensive answer for the user.

Your final answer to the user must include:  
1\.  \*\*Technical Trend:\*\* The 5-day price trend.  
2\.  \*\*Momentum:\*\* If the price is above/below the 20-day SMA.  
3\.  \*\*News Context:\*\* The \*most important\* news headline or snippet that seems to be driving the price.

\*\*IMPORTANT:\*\* Do not give any financial advice, predictions, or suggestions to buy, hold, or sell.”

## **Setup Prerequisites**

To run this workflow, you will need:

1. **n8n (Hosted using ngrok)**  
2. **Telegram Bot Credentials**:  
   * A Telegram Bot API Key from BotFather.  
3. **Alpha Vantage API Key**:  
   * A free API key from [Alpha Vantage](https://www.google.com/search?q=https.google.com/search?q%3Dhttps://www.alphavantage.co/support/%2523api-key).  
4. **SerpApi API Key**:  
   * A free API key from [SerpApi](https://serpapi.com/) for Google Search results.  
5. **Google Gemini API Key**:  
   * An API key from [Google AI Studio](https://aistudio.google.com/app/apikey).

## **Error Handling**

This workflow can be made more robust by checking for bad API responses. If a user sends a fake ticker (e.g., "FAKE"), the Alpha Vantage API will return an {"Error Message": ...}.

You can handle this by adding an **IF node** after each of the three HTTP Request nodes.

1. **Condition:** {{ $json\["Error Message"\] }} \-\> Is Empty  
2. **true path (Happy Path):** Connects to the Edit Fields node to proceed.  
3. **false path (Error Path):** Connects to a separate Send a text message (Telegram) node that sends a friendly error message back to the user (e.g., "Sorry, I couldn't find that ticker.").

## **Extensibility (Future Ideas)**

* **Google Sheets Trigger:** Add a second trigger (like the Review Agent) that watches a Google Sheet. You could add 10 tickers to the sheet and have the bot run all of them in a batch.  
* **Log to Database:** Use the Google Sheets or Postgres node as a tool (like we discussed) to log every analysis the AI performs.  
* **More Indicators:** Add more parallel branches to fetch other technical indicators like RSI or MACD and feed them all to the AI for an even deeper analysis.  
* **Error Notifications:** On the error path, send a push notification (like ntfy.sh) to your *own* phone so you know the bot failed.
**PPT** https://github.com/saivignesh060/abcd-agentic-training-vnr-vignesh/blob/8df9c77fb3c4a57c3bc9a502c2213e8cfae9d20d/agentic%20ai%20project/Stock%20Analyst%20Agent%20Presentation.pptx
**Demo Video**  https://github.com/saivignesh060/abcd-agentic-training-vnr-vignesh/blob/8df9c77fb3c4a57c3bc9a502c2213e8cfae9d20d/agentic%20ai%20project/DEMO.wmv
