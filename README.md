# Pawn Best Practices 🚀

Essential Pawn best practices, high-performance optimization techniques, and architectural guidelines tailored for both **SA-MP** and the modern **open.mp** framework.

This repository is designed to help developers transition from legacy SA-MP habits to modern, efficient Pawn scripting. It is also fully compatible with AI coding agents.

## 🤖 Using with AI Agents (skills.sh / Cline / Windsurf)

If you are using an AI coding assistant, you can integrate these standards directly into your workspace. 

1. Copy the `SKILL.md` file from this repository.
2. Place it into your agent's skills directory (e.g., `.agents/skills/pawn-best-practices/SKILL.md`).
3. Your AI will now automatically follow these best practices when writing or refactoring `.pwn` files.

## 📚 What's Inside?

This guide covers the following key areas:

* **Naming & Consistency:** Standardized variable and macro naming conventions.
* **Advanced Optimization:** Techniques to reduce server lag (e.g., `foreach`, memory management, high-performance math).
* **Modern Syntax:** Clean logic using modern Pawn features.
* **Architectural Standards:** Best practices for hooks (`y_hooks`), module isolation, and identity management.
* **Database & MySQL:** Guidelines for threaded queries and preventing SQL injections (`mysql_format`).
* **Streamer Plugin Strategy:** Knowing when (and when not) to use dynamic objects.
* **Anti-Patterns:** Common mistakes that kill server performance.

## 🛠️ Quick Example

**Legacy (Slow & Bloated):**
```c
new msg[1024];
for(new i = 0; i < MAX_PLAYERS; i++) {
    if(IsPlayerConnected(i)) {
        // Logic
    }
}
````

**Modern (Fast & Clean):**

```c
// String declared only when needed. Loop uses foreach.
foreach(new i : Player) {
    // Logic
}
```

## 📄 License

This project is licensed under the [MIT License](https://www.google.com/search?q=LICENSE) - see the https://www.google.com/search?q=LICENSE file for details. You are free to use, modify, and distribute these standards in your own projects.
