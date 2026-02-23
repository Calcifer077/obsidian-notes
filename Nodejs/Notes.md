How to take input from the command line?
The simplest way is using the `readLine` module, which provides an interface for reading data from a readable stream one line at a time.

**`import { createInterface } from "node:readline";`**

**`const readline = createInterface({`**
  **`input: process.stdin,`**
  **`output: process.stdout,`**
**`});`**

**`readline.question("What is your name? ", (name) => {`**
  **`console.log(Hello ${name}!);`**
  **`readline.close();`**
**`});`** 

For more complex scenarios, you can use the promise-based version:
**`import { createInterface } from "node:readline/promises";`**

**`import { stdin as input, stdout as output } from "node:process";`**

**`const readline = createInterface({ input, output });`**

**`const answer = await readline.question("What is your name? ");`**

**`console.log(Hello ${answer}!);`**
**`readline.close();`**


