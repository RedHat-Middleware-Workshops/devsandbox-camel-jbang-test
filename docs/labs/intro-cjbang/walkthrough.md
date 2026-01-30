# Camel JBang - First steps

Familiarise yourself with the prototyping tool helping you to run, iterate and test your Camel applications.

*Camel JBang* is a command-line interface (CLI) that accelerates prototyping by letting developers quickly create, validate, and tweak flows without complex setups, making it perfect for rapid experimentation.

<img src="images/camel-jbang.png" alt="Camel JBang" width="40%" />

This chapter is recommended to understand how *Camel JBang* helps you work fast with *Apache Camel*.

---

## Camel JBang at play

When developers are tasked to deliver a solution, prototyping and experimentation are crucial to finding the best path forward, especially when adopting new, unfamiliar tools and techniques.

*Camel JBang* is one of those key prototyping tools.

<img src="images/camel-jbang.png" alt="Camel JBang" width="40%" />

This chapter is an introductory taste of *Camel JBang* showing you a basic group of commands. Hopefully you’ll get to see how much you can sharpen your axe when making a good use of *Camel JBang*.

### Find your terminal

The first time, your `lab` directory is empty:

Example:

```
📁 workshop
   📁 lab    (empty)
```

For this exercise, work from your terminal. Find it as shown below.

<img src="images/crw-terminal.png" alt="Dev Spaces terminal" width="100%" />

The following steps will ask you to run commands. Run them in your terminal.

### Playing a game with Camel JBang

Playing is a great way to learn. Here we’ll use a small game to discover a few things *Camel JBang* can do.

> **Note:** Later we’ll go deeper into the core of the workshop.

Pretend we’re organizing a tennis tournament. First, **player1**:

Copy and paste the following into your terminal to create the `player1.yaml` file:

> **Note:** If your terminal asks to confirm the paste, you can tick “✓ Do not ask me again” and click **Paste**.

<img src="images/paste-multiline.jpg" alt="Paste multiline" width="30%" />

```bash
cat <<EOF > player1.yaml
- route:
    id: player1
    from:
      uri: file:court/side1
      parameters:
        delete: true
      steps:
        - log:
            message: (ping)⋅⋅⋅━━━► 🥎
        - delay:
            constant: "{{pace:3000}}"
        - to:
            uri: file:court/side2
EOF
```

**Note:** player1 is on one side of the court and hits every ball. When hitting, the ball crosses the court (at a given pace) to the other side.

Run this to get player1 on the court:

```bash
camel run player1.yaml
```

**Note:** There are no dependency descriptors or extra files—*Camel JBang* reads `player1.yaml` and runs the route.

In your terminal you’ll see player1 ready. Example output:

```text
2025-10-26 12:40:32.210 INFO 2243 --- [           main] e.camel.impl.engine.AbstractCamelContext : Apache Camel 4.15.0 (player1) started in 242ms ...
```

In your folder you’ll see where player1 is waiting for balls. Example:

```text
❯ lab ❯ court/side1
```

### Throw a ball

Player1 is ready. Give them a 🥎.

From the terminal’s top-right corner, click **+** to open a second terminal:

<img src="images/terminal-plus.jpg" alt="Terminal plus" width="30%" />

In the new terminal, use *Camel JBang* to send a ball to player1:

**Tip:** The `send` command is useful for testing: you can send balls to routes and see what happens.

```bash
camel cmd send player1 --header CamelFileName=🥎
```

Switch back to the first terminal where player1 is running:

<img src="images/terminal-one.jpg" alt="Terminal one" width="25%" />

You should see player1 hitting the ball to the other side. Example output:

```text
2025-10-26 12:50:12.951 INFO 2243 --- [e://court/side1] player1.yaml:8 : (ping)⋅⋅⋅━━━► 🥎
```

Check the court: the ball should be on the other side:

<img src="images/court-side2-ball.jpg" alt="Ball on side 2" width="8%" />

**Note:** With only one player, the ball ends up on the other side and stays there.

Before continuing, stop player1 with **Ctrl+C**.

### The ball boy

Next we add the ball boy to keep the court tidy.

Copy and paste this into your terminal to create `the-ballboy.yaml`:

```bash
cat <<EOF > the-ballboy.yaml
- route:
    id: ballboy
    from:
      uri: timer:check-court
      steps:
        - filter:
            groovy:
              expression: >-
                def court = ['court/side1/🥎', 'court/side2/🥎'];
                def file = court.find { new File(it).exists() }?.with { new File(it) };

                if (file?.exists() && (System.currentTimeMillis() - file.lastModified()) > {{wait:6000}} )
                  return file.delete();
                else
                  return false;
            steps:
              - log:
                  message: "🥎 cleared, court ready."
EOF
```

**Note:** In *Camel*, some tasks need a bit of logic. The ball boy uses *Groovy* to check both sides of the court and remove any ball that has been sitting for too long (6 seconds).

When you’re ready, start everything with:

```bash
camel run *
```

**Note:** *Camel JBang* supports a `*` wildcard when you have multiple route files. It also picks up property files. Optional example file `game.properties` (for reference only; you don't run this in the terminal):

```properties
# Ball pace (millis) when players hit. Lower value is faster.
pace=3000

# Ballboy waits (millis) before picking up the ball.
wait=6000
```

In the logs you should see the ball boy at work. Example output:

```text
2025-10-26 18:23:53.558 INFO 20299 --- [r://check-court] the-ballboy.yaml:17 : 🥎 cleared, court ready.
```

On the court, the ball will have been cleared:

<img src="images/court-clear.jpg" alt="Court clear" width="8%" />

Let player1 practice a bit more, then press **Ctrl+C** to stop.

### Set up the tournament

Practice is over; time for a real game.

You need a second player. Here is **player2**:

Copy and paste to create `player2.yaml`:

```bash
cat <<EOF > player2.yaml
- route:
    id: player2
    from:
      uri: file:court/side2
      parameters:
        delete: true
      steps:
        - log:
            message: "      🥎 ◄━━━⋅⋅⋅(pong)"
        - delay:
            constant: "{{pace:3000}}"
        - to:
            uri: file:court/side1
EOF
```

**Note:** Each player does one job: receive from one side and send to the other.

To run a proper “tournament”:

1. **Stop any running routes**  
   Press **Ctrl+C** if something is still running.

2. **Start the ball boy first (in the background):**

   ```bash
   camel run the-ballboy.yaml --background
   ```  
   The `--background` flag runs the process detached.

3. **Start both players (in the background, with live reload):**

   ```bash
   camel run player1.yaml --background --dev
   ```

   ```bash
   camel run player2.yaml --background --dev
   ```  
   The `--dev` flag enables live code reload (e.g. when you change the route).

4. **Check that everyone is running:**

   ```bash
   camel ps
   ```  
   **Note:** `ps` shows which routes are running and their status.

5. **Watch the logs:**

   ```bash
   camel log
   ```  
   **Note:** `log` aggregates output from all running routes. You can also follow a single route, e.g. `camel log player1`.

The ball boy and both players should be ready.

### Let the game begin

Player1 serves. Send a ball:

```bash
camel cmd send player1 --header CamelFileName=🥎
```

In the terminal you should see both players hitting the ball. Example output:

```text
player1    | 2025-10-26 19:01:36.006 INFO 22363 --- [e://court/side1] player1.yaml:8 : (ping)⋅⋅⋅━━━► 🥎
player2    | 2025-10-26 19:01:39.106 INFO 22578 --- [e://court/side2] player2.yaml:8 :       🥎 ◄━━━⋅⋅⋅(pong)
player1    | 2025-10-26 19:01:42.509 INFO 22363 --- [e://court/side1] player1.yaml:8 : (ping)⋅⋅⋅━━━► 🥎
player2    | 2025-10-26 19:01:45.610 INFO 22578 --- [e://court/side2] player2.yaml:8 :       🥎 ◄━━━⋅⋅⋅(pong)
...
```

That’s a rally.

You can also see the ball file moving on disk (court/side1 and court/side2):

<img src="images/court-rally.jpg" alt="Court rally" width="18%" />

Player1 stops to tie their laces. Stop their route:

```bash
camel cmd stop-route --id player1
```

You’ll see player2 hit once more, player1 stop, and the ball boy clear the ball. Example output:

```text
player2    | ... :       🥎 ◄━━━⋅⋅⋅(pong)
player1    | ... : Stopped player1 (file://court/side1)
the-ballboy| ... : 🥎 cleared, court ready.
```

#### Game statistics

Inspect route stats:

```bash
camel get route
```

Example output:

```text
  PID   NAME         ID       FROM                            REMOTE  STATUS    AGE   COVER  MSG/S  TOTAL  FAIL  INFLIGHT  MEAN  MIN   MAX   LAST  DELTA  SINCE-LAST
 22140  the-ballboy  ballboy  timer://check-court                     Started   6m3s    2/2   1.00    363     0         0     2     0   841     0      0            0s/0s/-
 22363  player1      player1  file://court/side1?delete=true    x     Stopped           3/3   0.00     25     2         0  2001  2001  2007  2001      0  1m31s/1m29s/1m41s
 22578  player2      player2  file://court/side2?delete=true    x     Started  5m21s    3/3   0.00     23     0         0  2001  2001  2012  2001      0      1m29s/1m27s/-
```

**Note:** player1 is stopped but still listed; the ball boy and player2 keep running.

#### The game resumes

Player1 is ready again. Start the route:

```bash
camel cmd start-route --id player1
```

Player2 serves. Send a ball:

```bash
camel cmd send player2 --header CamelFileName=🥎
```

The rally continues.

#### Game conditions

Suppose the weather changes: wind, clouds, a bit of rain. We can change the “game conditions” in the YAML.

While the rally is going, edit the files:

- In **`player1.yaml`**: change the log message to `p1ng` and the delay to `{{pace:4000}}` (slower).
- In **`player2.yaml`**: change the log message to `p0ng` and the delay to `{{pace:4000}}` (slower).

**Note:** With `--dev`, *Camel JBang* reloads the routes and the new behavior takes effect without restarting.

In the logs you’ll see reload messages. Example output:

```text
player1    | ... RouteWatcherReloadStrategy : Routes reloaded summary (total:1 started:1)
player1    | ... RouteWatcherReloadStrategy : Started player1 (file://court/side1) (source: player1.yaml:4)
```

Then the slower “p1ng” / “p0ng” messages. Example output:

```text
player2    | ... :       🥎 ◄━━━⋅⋅⋅(p0ng)
player1    | ... : (p1ng)⋅⋅⋅━━━► 🥎
...
```

### Abrupt end

Player1 slips on the wet court and has to leave the game. Stop that route:

```bash
camel stop player1
```

**Tip:** Run `camel ps` to confirm player1 is no longer in the list.

Then stop everything:

```bash
camel stop
```

**Note:** With no name, *Camel JBang* stops all running Camel processes.

That’s the end of this match—but you can run the game again anytime.

🥎🥎🥎  
Keep exploring the workshop for more.  
🥎🥎🥎

---

## Clean up your lab folder

Keep your `lab` folder clean for the next exercises.

Run:

```bash
rm -r *
```

Your `lab` folder should contain no files or directories.

---

## Closing words

This was a short introduction to working quickly with a few *Camel* route files. *Camel JBang* has many more commands to help with development.

Quick recap of what you used:

**Running Camel**

- Run a single file: `camel run <file>.yaml`
- Run all: `camel run *`
- Run in the background: `camel run <file>.yaml --background`
- Run with live reload: `camel run <file>.yaml --background --dev`

**Troubleshooting and monitoring**

- List running instances: `camel ps`
- Follow logs: `camel log` or `camel log <route-id>`
- Start/stop routes: `camel cmd start-route --id <id>`, `camel cmd stop-route --id <id>`
- Route details: `camel get route`
- Send a message: `camel cmd send <route-id> --header CamelFileName=🥎`

**Stopping Camel**

- Stop one instance: `camel stop <name>`
- Stop all: `camel stop`

Continue with other chapters in the workshop to see what *Camel JBang* can do beyond this intro, and how it fits with other tools for working quickly with *Apache Camel*—from initial concepts to deployment on platforms like OpenShift.
