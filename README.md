# Dynamic UI Element Selection in Tricentis Tosca

While automating an e-commerce flow (like selecting a product color between White, Black, Green etc.), normally we add a `{RND}` function either in `InnerText|OuterText|XPath` to randomize the selection. The expectation is with each run, the engine should choose a different color.

I used to do this too, but then I observed a major flaw in it. It lacked predictability. On using the `{RND}` function, the same random number got picked more than once, hence the same color got picked in consecutive execution runs, leaving the other color variants untested during a cycle.

To solve this and guarantee 100% sequential test coverage without maintaining a messy external spreadsheet, I built a self-initializing rolling counter directly inside Tosca using a bit of modulo math.

---

## 🛠️ THE SETUP

You only need one **TBox Set Buffer** step with two quick rows inside your TestCase:

| Buffer Name | Value | ActionMode |
| :--- | :--- | :--- |
| **Color?** | `0` | `Input` |
| **Color** | `{MATH[({B[Color]}+0)%3+1]}` | `Input` |

### Why this works beautifully:
The magic is that tiny question mark (`Color?`). It tells Tosca: *“Only set this buffer to 0 if it doesn't exist yet.”* 
On your first run ever, it creates the buffer at `0`, and the math row turns it into a `1`. On every run after that, Tosca completely skips the first row, reads the existing number, and loops it sequentially: **1 ➡️ 2 ➡️ 3 ➡️ 1...** forever.

---

## 🔍 DYNAMIC XPATH CONFIGURATION

To feed this number straight to the browser without breaking the HTML parser, wrap the buffer statement in double quotes inside your Module Attribute’s **XPath** parameter:

```xpath
id('color-squares-11')/li["{B[Color]}"]/label/span/span
```

---

## 💡 THE BIG WINS
* **Zero Maintenance:** It’s 100% self-contained. You don't have to touch your Project Settings or build local variables. 
* **Pipeline Ready:** You can hand this script off to a teammate or push it to a CI/CD agent, and it will run instantly without throwing a "Buffer not found" error.
* **Guaranteed Coverage:** If you trigger your test suite 3 times, you are guaranteed to test all 3 product variations exactly once.
---

## 📸 SCREENSHOTS SEQUENCE

### 1️⃣ E-Commerce Site Selection
![E-Commerce Site](E-Commerce%20Site.png)

### 2️⃣ The Exact Module Attribute Configuration
![Module Attribute](The%20exact%20Module%20attribute.png)

### 3️⃣ TestCase Layout: Step 1 (Conditional Buffer Initialization)
![Test Step 1](The%20test%20step%201.png)

### 4️⃣ TestCase Layout: Step 2 (Modulo Math Expression)
![Test Step 2](The%20test%20step%202.png)

### 5️⃣ TestCase Layout: Step 3 (Dynamic Steering Selection)
![Test Step 3](The%20test%20step%203.png)


