# Python Logging
### A meditation on **silent** failures

^ Quick intro: Jess Unrein. Platform Engineer with Sprout. Former bootcamp graduate and jazz pianist.

^ The Zen of Python teaches us that errors should never pass silently. 

^ Does not make a disticntion between errors and failures, which seems pretty important to me.

---

# [fit] Errors vs. **Failures**

^ An error is something that causes your code to barf. Unless you've explicitly silenced something, it should let you know in a loud way.

^ However, not every failure is an error. Just because your code didn't scream at you doesn't mean everything functioned as intended.

^ Good logging and monitoring can help you detect **failures** that won't show up in a stack trace

---

## Shouldn't I just test my code?

^ Sure. In a perfect world where we all write perfect code. Why didn't I think of that?

---

# [fit] What is **logging**?

^ Logging is record keeping for your app

^ It gives critical information about errors you might see, behaviors you don't see but should, and gives you clues to detect problems that you might not otherwise run across.

---

Anecdote about SQLAlchemy change --> email failures

---