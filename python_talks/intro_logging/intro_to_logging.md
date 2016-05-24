# Python Logging
### A meditation on **silent** failures

---

Intro goes here

---

# [fit] What is logging?

---

# [fit] Errors vs. **Failures**

^ An error is something that causes your code to barf. Unless you've explicitly silenced something, it should let you know in a loud way.

^ However, not every failure is an error. Just because your code didn't scream at you doesn't mean everything functioned as intended.

^ Good logging and monitoring can help you detect **failures** that won't show up in a stack trace

---

## Shouldn't I just test my code?

^ Sure. In a perfect world where we all write perfect code. Why didn't I think of that?

---

Anecdote about SQLAlchemy change --> email failures

---