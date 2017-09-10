# statebox
> Orchestrate Node functions using Amazon States Language

## Useful links

* https://aws.amazon.com/step-functions/
* http://docs.aws.amazon.com/step-functions/latest/dg/concepts-amazon-states-language.html
* http://docs.aws.amazon.com/step-functions/latest/apireference/API_SendTaskSuccess.html


## <a name='install'></a>Install
```bash
$ npm install statebox --save
```

## <a name='usage'></a>Usage

```javascript
const Statebox = require('statebox')
const statebox = new Statebox()

// STEP 1:
// Register some 'module' resources (i.e. Javascript 
// functions) that state machines can refer to...
// -------------------------------------------------

statebox.createModuleResource(
  'add',
  function addTwoNumbers (event, context) {
    this.sendTaskSuccess(event.number1 + event.number2)
  }
)
statebox.createModuleResource(
  'subtract',
  function subtractOneNumberFromAnother (event, context) {
    this.sendTaskSuccess(event.number1 - event.number2)
  }
)

// STEP 2:
// Create a new 'calculator' state machine
// using Amazon States Language...
// ---------------------------------------

const info = statebox.createStateMachine(
  'calculator',
  {
    Comment: 'A simple calculator',
    StartAt: 'OperatorChoice',
    States: {
      OperatorChoice: {
        Type: 'Choice',
        Choices: [
          {
            Variable: '$.operator',
            StringEquals: '+',
            Next: 'Add'
          },
          {
            Variable: '$.operator',
            StringEquals: '-',
            Next: 'Subtract'
          }
        ]
      },
      Add: {
        Type: 'Task',
        InputPath: '$.numbers',
        Resource: 'module:add', // i.e something added via createModuleResource()
        ResultPath : '$.result',
        End: true
      },
      Subtract: {
        Type: 'Task',
        InputPath: '$.numbers',
        Resource: 'module:subtract',
        ResultPath : '$.result',
        End: true
      }
    }
  }
)

// STEP 3:
// Create a new execution on a state machine
// -----------------------------------------

statebox.startExecution(
  {
    numbers: {
      number1: 3,
      number2: 2
    },
    operator: '-'
  },  // input
  'calculator', // state machine name
  {}, // options
  function (err, result) {
    // Result object
    // -------------
    //  {
    //    executionName: '01e1e288-9533-11e7-8fec-54d168e2e610',
    //    ctx: {
    //      numbers: {
    //        number1: 3,
    //        number2: 2
    //      },
    //      operator: '-'
    //    },
    //    currentStateName: 'OperatorChoice',
    //    stateMachineName: 'calculator',
    //    status: 'RUNNING',
    //    startDate: '2017-09-10T09:40:22.589Z'
    //  }
  }
)

// --            ~~ Passing of time ~~~
// --        TODO: Implement Statebox events

// STEP 4:
// Look at the results...
// -----------------------------------------
statebox.describeExecution(
  '01e1e288-9533-11e7-8fec-54d168e2e610',
  function (err, result) {
    //  Result object
    //  -------------
    // {
    //   executionName: '01e1e288-9533-11e7-8fec-54d168e2e610',
    //   ctx: {
    //     numbers': {
    //       number1: 3,
    //       number2: 2
    //     },
    //     operator: '-',
    //     result: 1 <--- The important bit :-)
    //   },
    //   currentStateName: 'Subtract',
    //   stateMachineName: 'calculator',
    //   status: 'SUCCEEDED',
    //   startDate: '2017-09-10T09:59:50.711Z'
    // }
  }
)
    
```

## <a name='test'></a>Testing

```bash
$ npm test
```

## <a name='license'></a>License
[MIT](https://github.com/wmfs/tymly/packages/statebox/blob/master/LICENSE)
