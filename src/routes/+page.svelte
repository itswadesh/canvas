<script>
import { onMount } from 'svelte'
import Konva from 'konva'
let container
let json=''
let state
let layer
let stage
let appHistoryStep = 0
let appHistory = [state];
state = [createYoda({ x: 50, y: 50 })];
 let possibleFilters = [''];
let width = window.innerWidth;
let height = window.innerHeight;
onMount(() => {
      stage = new Konva.Stage({
        container: 'container',
        width: width,
        height: height,
      });
      layer = new Konva.Layer();
      stage.add(layer);
      create(state);
})
function saveStateToHistory(state) {
  appHistory = appHistory.slice(0, appHistoryStep + 1);
  appHistory = appHistory.concat([state]);
  appHistoryStep += 1;
}
function create() {
  layer.destroyChildren();
  state && state.forEach((item, index) => {
    var node = new Konva.Image({
      draggable: true,
      name: 'item-' + index,
      // make it smaller
      scaleX: 0.5,
      scaleY: 0.5,
    });
    layer.add(node);
    node.on('dragend', () => {
      // make new state
      state = state.slice();
      // update object data
      state[index] = Object.assign({}, state[index], {
        x: node.x(),
        y: node.y(),
      });
      // save it into history
      saveStateToHistory(state);
      // don't need to call update here
      // because changes already in node
    });

    node.on('click', () => {
      // find new filter
      var oldFilterIndex = possibleFilters.indexOf(state[index].filter);
      var nextIndex = (oldFilterIndex + 1) % possibleFilters.length;
      var filter = possibleFilters[nextIndex];

      // apply state changes
      state = state.slice();
      state[index] = Object.assign({}, state[index], {
        filter: filter,
      });
      // save state to history
      saveStateToHistory(state);
      // update canvas from state
      update(state);
    });

    var img = new window.Image();
    img.onload = function () {
      node.image(img);
      update(state);
    };
    img.src = item.src;
  });
  update(state);
}
function update() {
  state && state.forEach(function (item, index) {
    var node = stage.findOne('.item-' + index);
    node.setAttrs({
      x: item.x,
      y: item.y,
    });

    if (!node.image()) {
      return;
    }
    if (item.filter === 'blur') {
      node.filters([Konva.Filters.Blur]);
      node.blurRadius(10);
      node.cache();
    } else if (item.filter === 'invert') {
      node.filters([Konva.Filters.Invert]);
      node.cache();
    } else {
      node.filters([]);
      node.clearCache();
    }
  });
}
function createObject(attrs) {
  return Object.assign({}, attrs, {
    x: 0,
    y: 0,
    src: '',
  });
}
function createYoda(attrs) {
  return Object.assign(createObject(attrs), {
    src: 'https://ik.imagekit.io/3wzatecz51w3i/zapvi/product/20220831/2664-uwllc20wpd7x.jpg?tr=w-416,h-480:w-416,h-480',
  });
}
function createDarth(attrs) {
  return Object.assign(createObject(attrs), {
    src: 'https://ik.imagekit.io/3wzatecz51w3i/zapvi/product/20220831/2660-nbivebd9leuf.jpg?tr=w-416,h-480:w-416,h-480',
  });
}
function createYoda1(){
    // create new object
    state.push(
      createYoda({
        x: 100,
        y: 100,
      })
    );
    // recreate canvas
    create(state);
  }

function createDarth1(){
    // create new object
    state.push(
      createDarth({
        x: 100,
        y: 100,
      })
    );
    // recreate canvas
    create(state);
  }

function  undo () {
  if (appHistoryStep === 0) {
    return;
  }
  appHistoryStep -= 1;
  state = appHistory[appHistoryStep];
  // create everything from scratch
  create(state);
}
function  redo () {
        if (appHistoryStep === appHistory.length - 1) {
          return;
        }
        appHistoryStep += 1;
        state = appHistory[appHistoryStep];
        // create everything from scratch
        create(state);
      }
</script>
<button on:click="{createYoda1}">Create yoda</button>
<button on:click="{createDarth1}">Create darth</button>
<button on:click="{undo}">Undo</button>
<button on:click="{redo}">Redo</button>
<div bind:this="{container}" id="container"></div>
<!-- <button on:click="{toJSON}">export</button> -->
<br />
<br />
<br />
<code class="code">
	{json}
</code>