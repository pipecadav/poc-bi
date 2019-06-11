# JS API

## Get embed URL within Domo

Only Private embed supported

```
  <iframe src="https://example.domo.com/embed/card/12345" width="600" height="600"></iframe>
```

## Add a message listener to window

When successfully loaded, the Domo Everywhere iframe will call `postMessage` on
its parent window with a `MessagePort` used for interaction.

To subscribe to the message use `window.addEventListener('message')`.


```js
let port;

window.addEventListener('message', e => {
  if (e.origin !== 'https//example.domo.com') return;
  if (!e.ports[0]) return;
  port = e.ports[0];
});
```

## Make a request on the MessagePort

After receiving the MessagePort, you're ready to interact with Domo Everywhere.
Make a call to one of the available APIs using JSON-RPC 2.0 format.

```js
port.postMessage({
  id: 'setFilters123',
  jsonrpc: '2.0',
  method: '/v1/filters/apply',
  params: {
    filters: [{
      column: 'year',
      operator: 'GREATER_THAN',
      values: ['1990']
    }]
  }
});
```

Responses will be sent on the `port` echoing the original `id`

```js
port.onmessage = e => {
  console.log(e.data); // {id: 'setFilters123', ...}
}
```

## APIs

The following APIs are available as the `method` field

### /v1/filters/apply

Apply an array of filters to the current chart. Every call will replace all current filters.


#### Params:

```js
type Params = {
  filters: Filter[]
}

type Filter = {
  column: string,
  operator: Operator,
  values: string[],
}

type Operator =
  'GREATER_THAN' |
  'GREAT_THAN_EQUALS_TO' |
  'LESS_THAN' |
  'LESS_THAN_EQUALS_TO' |
  'EQUALS' |
  'NOT_EQUALS' |
  'IN' |
  'NOT_IN';
```

#### Response

```js
undefined
```


## Full Example

```html

<!DOCTYPE HTML>

<iframe src="http://localhost:3000/embed/card/864587738" width="600" height="600"></iframe>
<form id="filterform">
  <input name="column" placeholder="column" />
  <select name="operator">
    <option>GREATER_THAN</option>
    <option>GREAT_THAN_EQUALS_TO</option>
    <option>LESS_THAN</option>
    <option>LESS_THAN_EQUALS_TO</option>
    <option>EQUALS</option>
    <option>NOT_EQUALS</option>
    <option>IN</option>
    <option>NOT_IN</option>
  </select>
  <input name="filterValue" placeholder="value" />
  <button>Save</button>
</form>
<script>
(function() {
  let port;
  const onPortMessage = e => {
    console.log(e);
  };

  // https://www.jsonrpc.org/specification

  window.addEventListener("message", e => {
    // Check origin
    if (!e.ports[0]) return;
    port = e.ports[0];
    port.onmessage = onPortMessage;
  });

  const applyFilters = (filters = []) =>
    port.postMessage({
      id: 'setFilters123',
      jsonrpc: '2.0',
      method: '/v1/filters/apply',
      params: {
        filters
      }
    });

  const onSubmit = e => {
    e.preventDefault();
    // read the values off of the form inputs
    const [column, operator, filterValue] = [...e.target].map(
      element => element.value
    );

    // Apply a single filter to the embed
    applyFilters([{ column, operator, values: [filterValue] }]);
  };

  const form = document.querySelector("#filterform");
  form.addEventListener("submit", onSubmit);
})();
</script>
```

## Example with Multiple Ports

``` js

const ports = [];
     const onPortMessage = e => {
       console.log(e);
     };
 
     // https://www.jsonrpc.org/specification
     window.addEventListener("message", e => {
       console.log('received message', e);
       // Check origin
       if (!e.ports[0]) return;
       const port = e.ports[0];
       port.onmessage = onPortMessage;
       ports.push(port);
     });
 
     const applyFilters = (filters = []) =>
       ports.forEach(port => port.postMessage({
         id: 'setFilters123',
         jsonrpc: '2.0',
         method: '/v1/filters/apply',
         params: {
           filters
         }
       }));
 
     const select = document.querySelector("#filterselect");
     select.addEventListener("change", function() {
       console.log('value = ' + select.value);
       switch(select.value) {
         case "All":
           applyFilters();
           break;
         default:
           applyFilters([{"column": "State", "operator": "IN", "values": [select.value]}]);
       }
     });


```