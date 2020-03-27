Child processes can be created and communicated with using the ```spawn``` function in Node JS

https://nodejs.org/api/child_process.html#child_process_child_process_spawn_command_args_options

1. Create a new child process using spawn method. We can optionally pass command line arguments via the array as shown below
```js
const { spawn } = require('child_process');
const childPs = spawn('hello.py', ['--name', 'Sudhir']);
```

2. Send data to child process by writing to the *stdin* stream. We can signal the end of stream using the *end* method.
```js
childPs.stdin.write("Some application data to the child process, may be json")
childPs.stdin.end();
```

3. Read data from child process using the *stdout* data listener. We can get to know the end of stream via the *close* event
```js
childPs.stdout.on('data', (data) => {
  console.log('recieved data from child process');
  console.log(data);
});

childPs.stdout.on('close', (code) => {
  console.log('child process exited with code ' + code);
});
```

4. An example that implements ```ps ax | grep ssh``` command by spawning two child processes is shown below for a practical example
```js
const { spawn } = require('child_process');
const ps = spawn('ps', ['ax']);
const grep = spawn('grep', ['ssh']);

ps.stdout.on('data', (data) => {
  grep.stdin.write(data);
});

ps.stderr.on('data', (data) => {
  console.error(`ps stderr: ${data}`);
});

ps.on('close', (code) => {
  if (code !== 0) {
    console.log(`ps process exited with code ${code}`);
  }
  grep.stdin.end();
});

grep.stdout.on('data', (data) => {
  console.log(data.toString());
});

grep.stderr.on('data', (data) => {
  console.error(`grep stderr: ${data}`);
});

grep.on('close', (code) => {
  if (code !== 0) {
    console.log(`grep process exited with code ${code}`);
  }
});

```

4. We can ```pipe``` a stream to child process stdin as shown in the example below taken from [here](https://blog.cloudboost.io/node-js-child-process-spawn-178eaaf8e1f9)
```js
const { spawn } = require('child_process');

// Get the child from spawn with echo 
const child = spawn('wc');

// Pipe the input on the parent stdin to the child stdin
process.stdin.pipe(child.stdin);
// Listen to events from the stdout on the child

child.stdout.on('data', () => {
    console.log(`stdout:\n${data}`);
});
```

5. In case of messaging between 2 node js processes, *fork* can be used for convinient messaging as shown below taken from [here](https://www.freecodecamp.org/news/node-js-child-processes-everything-you-need-to-know-e69498fe970a/)
```js
// parent file
const { fork } = require('child_process');

const forked = fork('child.js');

forked.on('message', (msg) => {
  console.log('Message from child', msg);
});

forked.send({ hello: 'world' });
```

```js
// child process file
process.on('message', (msg) => {
  console.log('Message from parent:', msg);
});

let counter = 0;

setInterval(() => {
  process.send({ counter: counter++ });
}, 1000);
```

