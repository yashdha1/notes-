
/init -> inititlize and scrape entire project with the claude. (create ./claude)

##### ADDING MCP TO THE CLAUDE
- Adding the MCP servers to the claude configurations. using the ./claude/settings.local.json 
##### Adding claude as a github action. 
- /install-github-app -> for the github integrtaions in the claude.. 
	- either on PR
	- or on the @mention action. 

##### Hooks in the claude

- runs before or after the claude does something either before or after the claude starts. 
	- code formatter after claude does shi .
	- stop claude from editiong some imp shit. 
	- log the changes made by claude in external files. 
	- run test 
	- block deprecated code usage. 

#### How Hooks Work

To understand hooks, let's first review the normal flow when you interact with Claude Code. When you ask Claude something, your query gets sent to the Claude model along with tool definitions. Claude might decide to use a tool by providing a formatted response, and then Claude Code executes that tool and returns the result. Hooks insert themselves into this process, allowing you to execute code just before or just after the tool execution happens.

![[Pasted image 20260529122324.png]]

Two of the most common hook types are below :-> 
- **PreToolUse hooks** - Run before a tool is called
- **PostToolUse hooks** - Run after a tool is called

--- 

### MCP 
adding mcp to the project using the 

[Explore the .claude directory - Claude Code Docs](https://code.claude.com/docs/en/claude-directory) 

> codemie-claude mcp add <product_name> <remote_executor> @name@latest

--- 
FRAMEWORK FOR THE prompting the agents  -> [AI Fluency Framework | Documentation, papers, presentations, and OER related to the AI Fluency Framework](https://aifluencyframework.org/) 

#### Topics 
- TEMPRATURE, ENCODERS DECODERS, TOKENIZATIONS.
- Invoke, streaming and batch.
- **Prompt Engineering** and **Prompt Evaluation** (prompt eval pipelines to test the users prompt among the tested and know correct output).
- Prompt Scorering/Evaluations pipelines. 

	> { Prompt -> eval dataset -> model -> Grader -> change prompt little bit and feed again. } * N -> then evaluate the metrics and do shit. 
- Wrap some of the structured things in the xml tag for better understanding for the model. 
- One Shot and Multi Shot prompting. 
