Child processes can be created and communicated with using the ```spawn``` function in Node JS

https://nodejs.org/api/child_process.html#child_process_child_process_spawn_command_args_options

* Create a new child process using spawn method. We can optionally pass command line arguments via the array as shown below
```js
// node js
const { spawn } = require('child_process');
const childPs = spawn('hello.py', ['--name', 'Sudhir']);
```

* Send data to child process by writing to the *stdin* stream. We can signal the end of stream using the *end* method.
```js
// node js
childPs.stdin.write("Some application data to the child process, may be json")
childPs.stdin.end();
```

* Read data from child process using the *stdout* data listener. We can get to know the end of stream via the *close* event
```js
// node js
childPs.stdout.on('data', (data) => {
  console.log('recieved data from child process');
  console.log(data);
});

childPs.stdout.on('close', (code) => {
  console.log('child process exited with code ' + code);
});
```

* We can ```pipe``` a stream to child process stdin as shown in the example below taken from [here](https://blog.cloudboost.io/node-js-child-process-spawn-178eaaf8e1f9)
```js
// node js
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

* An example that communicates with python script from node js using spawn is shown below taken from [here](https://www.sohamkamani.com/blog/2015/08/21/python-nodejs-comm/)
```js
//parent.js
var spawn = require('child_process').spawn,
    py    = spawn('python', ['compute_sum.py']),
    data = [1,2,3,4,5,6,7,8,9],
    dataString = '';

py.stdout.on('data', function(data){
  dataString += data.toString();
});
py.stdout.on('end', function(){
  console.log('Sum of numbers=',dataString);
});
py.stdin.write(JSON.stringify(data));
py.stdin.end();
```

```py
## compute_sum.py
import sys, json, numpy as np

#Read data from stdin
def read_in():
    lines = sys.stdin.readlines()
    #Since our input would only be having one line, parse our JSON data from that
    return json.loads(lines[0])

def main():
    #get our data as an array from read_in()
    lines = read_in()

    #create a numpy array
    np_lines = np.array(lines)

    #use numpys sum method to find sum of all elements in the array
    lines_sum = np.sum(np_lines)

    #return the sum to the output stream
    print lines_sum

# start process
if __name__ == '__main__':
    main()
```

* An example in which a python script communicates to node js process repeatedly using its output stream is shown below. This can be used for real time updates from child to parent process
```js
//parent.js
var spawn = require('child_process').spawn;

// python child process
var py = spawn('python', ['child.py'])

// listen for child output
py.stdout.on('data', function(data) {
    console.log(`${(new Date()).toLocaleString()}: ${data}`);
});

// listen for end of child
py.stdout.on('end', function() {
    console.log(`${(new Date()).toLocaleString()}: child process exited`);
});
```

```py
## child.py
import random
import time


def main():
    for pIter in range(1, 11):
        time.sleep(0.5)
        print('The output for iteration {0} is {1}'.format(
            pIter, random.randrange(10, 100)), flush=True)


# start process
if __name__ == '__main__':
    main()
```

* In case of messaging between 2 node js processes, *fork* can be used for convinient messaging as shown below taken from [here](https://www.freecodecamp.org/news/node-js-child-processes-everything-you-need-to-know-e69498fe970a/)
```js
// parent js file
const { fork } = require('child_process');

const forked = fork('child.js');

forked.on('message', (msg) => {
  console.log('Message from child', msg);
});

forked.send({ hello: 'world' });
```

```js
// child process js file
process.on('message', (msg) => {
  console.log('Message from parent:', msg);
});

let counter = 0;

setInterval(() => {
  process.send({ counter: counter++ });
}, 1000);
```
