# Project README

## Overview

This repository contains a set of Python notebooks focused on building conversational AI workflows and experimentations with large language models. The project demonstrates:

- Google Gemini integration with LangChain
- LangChain memory-enabled conversational chains


### add memory to build conversation chain

This notebook demonstrates a LangChain conversation chain with short-term memory.

Key elements:
- `ConversationBufferWindowMemory(k=3, return_messages=True)` to retain the last 3 turns
- `ChatPromptTemplate` with a `MessagesPlaceholder` for conversation history
- `RunnableLambda` + `RunnablePassthrough` to compose a chain that loads memory, formats prompts, and calls a model
- Example user interactions and memory save/load flow

Primary tools and libraries:
- `langchain.memory.ConversationBufferWindowMemory`
- `langchain_core.prompts.ChatPromptTemplate`
- `langchain_core.runnables.RunnableLambda`
- `langchain_core.runnables.RunnablePassthrough`

