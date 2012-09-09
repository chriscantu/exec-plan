exec-plan
=========

Provide the ability to run child process commands sequentially, with some fine-grained control, all while avoiding the
dread pyramid of doom (i.e., callback indentation) >.<

Examples
========

````javascript
/**
 * An example of the most basic usage.
 */

var ExecPlan = require('exec-plan').ExecPlan;
var execPlan = new ExecPlan();

execPlan.add('ls -la');
execPlan.add('grep "test" ./*');
execPlan.add('ps -ef');
execPlan.execute();
````

````javascript
/**
 * An example with more fine-grained control.
 */

var ExecPlan = require('exec-plan').ExecPlan;
var execPlan = new ExecPlan();

// attach event handlers to the events exposed by ExecPlan
execPlan.on('error', function (error, stderr) {
    console.log('an error happened in one of the steps in execution plan');
    console.log(error);  // the error object for the process that caused the problem
    console.log('the stderr for the process that caused the problem is ' + stderr);
});
execPlan.on('complete', function (stdout) {
    console.log('the entire execution plan finished, i.e., all child processes completed with no errors)');
    console.log('the stdout for the final step in the execution plan is ' + stdout);
});

// first, setup a vanilla set of commands
execPlan.add('ls -la');
execPlan.add('ps -ef');

// now, add a command that will include some 'pre logic' that will run before the command is executed, 
// but after previous command in the execution plan finished.
execPlan.add(function (stdout) {
    process.chdir('/tmp');  // run this logic before the command is executed
}, 'grep "test" ./*');
````