# 🧠 LLM Fundamentals — Simple Notes

---

# 🔤 1. Tokens

## 📌 Definition
Tokens = small pieces of text used by LLMs

---

## 🔍 Example
"I love AI" → ["I", " love", " AI"]

---

## 🔢 Key Points
- 1 token ≈ 3–4 characters  
- Input + output = total tokens  
- Pricing is based on tokens  

---

## ⚠️ Why Important
- More tokens → more cost  
- More tokens → slower response  

---

## 🧠 Summary
LLMs read tokens, not words

---

# 📊 2. Embeddings

## 📌 Definition
Embeddings = converting text into numbers (vectors)

---

## 🔍 Example
"cat" → [0.2, -0.4, 0.7]  
"dog" → [0.21, -0.39, 0.72]

👉 Similar meaning → similar vectors  

---

## 🧠 Use Cases
- Semantic search  
- RAG systems  
- Recommendations  

---

## 💡 Idea
LLM understands meaning, not exact words

---

## 🧠 Summary
Embeddings = meaning in numbers

---

# 🧠 3. Context Window

## 📌 Definition
Context window = how much text an LLM can read at once

---

## 📦 Includes
- System prompt  
- User input  
- Chat history  
- Retrieved data  

---

## ⚠️ If Exceeded
- Old data removed  
- Model forgets earlier context  

---

## 🔍 Example
User: My name is John  
User: What is my name?  

👉 Small context → forgets  
👉 Large context → remembers  

---

## 💡 Best Practices
- Keep prompts short  
- Send only relevant data  
- Limit history  

---

## 🧠 Summary
Context window = model’s memory

---

# 🧠 4. Prompt Engineering

## 📌 Definition
Prompt engineering = writing instructions for LLM

---

## 🧩 Types

### System Prompt
"You are a helpful tutor"

### User Prompt
"Explain AI simply"

### Few-Shot Prompting
Q: 2+2 → 4  
Q: 3+3 → 6  
Q: 5+5 → ?

---

## 💡 Best Practices
- Be clear and specific  
- Define output format  
- Use examples  

---

## 🔍 Example

❌ Bad:
"Tell me about AI"

✅ Good:
"Explain AI in 3 bullet points for beginners"

---

## 🧠 Summary
Better prompt = better output

---

# 🔥 Final Quick Summary

- Tokens → text pieces  
- Embeddings → meaning  
- Context → memory  
- Prompt → instructions  

---
