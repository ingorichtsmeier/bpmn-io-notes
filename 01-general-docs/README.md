# Neues BPMN-js Projekt
```
npm init
npm i --save bpmn-js
npm i --save jquery
```
```
npm i --save-dev webpack
```
-> version ^5.30.0
```
npm i --save-dev webpack-cli
```
-> version ^4.6.0
```
npm i --save-dev copy-webpack-plugin
```
-> version ^8.1.1
```
npm i --save-dev webpack-dev-server
```
-> version ^3.11.2
```
npm i --save-dev raw-loader
```

## webpack.config.js anlegen:
```javascript
const CopyWebpackPlugin = require('copy-webpack-plugin');

module.exports = {
  entry: {
    bundle: ['./app/app.js']
  },
  output: {
    path: __dirname + '/public',
    filename: 'app.js'
  },
  module: {
    rules: [
      {
        test: /\.bpmn$/,
        use: 'raw-loader'
      }
    ]
  },
  plugins: [
    new CopyWebpackPlugin({
      patterns: [
        { from: 'assets/**', to: 'vendor/bpmn-js', context: 'node_modules/bpmn-js/dist/' },
        { from: '**/*.{html,css}', context: 'app/' }
      ]
    })
  ],
  mode: 'development',
  devtool: 'source-map'
};
```

## dev-script in package.json einfügen:
```json
"dev": "webpack serve --content-base=public --open"
```

## Linting:
```
npm i eslint --save-dev
npm i eslint-plugin-bpmn-io --save-dev
```

### Konfiguration .eslintrc anlegen:
```json
{
  "extends": "plugin:bpmn-io/es6",
  "env": {
    "browser": true
  }
}
```
### Konfiguration .eslintignore anlegen:
```
public/
```
```
npm i npm-run-all --save-dev
``` 

### Linting script in package.json einfügen:
```json
"lint": "eslint .",
"start": "run-s dev"
```

## app/index.html anlegen:
```html
<!DOCTYPE html>
<html>

<head>
  <title>bpmn-js modeler demo</title>

  <link rel="stylesheet" href="vendor/bpmn-js/assets/diagram-js.css" />
  <link rel="stylesheet" href="vendor/bpmn-js/assets/bpmn-font/css/bpmn-embedded.css" />
  <link rel="stylesheet" href="css/app.css" />
</head>

<body>
  <div class="content" id="js-drop-zone">
    <div class="message intro">
      <div class="note">
        BPMN Diagram hierhin ziehen oder
        <a id="js-create-diagram" href>Diagramm erstellen</a>
      </div>
    </div>
    
    <div class="message error">
      <div class="note">
        <p> OOPS</p>
        <div class="details">
          <span>Grund</span>
          <pre></pre>
        </div>
      </div>
    </div>

    <div class="canvas" id="js-canvas"></div>
  </div>
  <ul class="buttons">
    <li>
      herunterladen
    </li>
    <li>
      <a id="js-download-diagram" href title="BPMN Diagramm herunterladen">
        BPMN Diagramm
      </a>
    </li>
    <li>
      <a id="js-download-svg" href title="Als Vektorgrafik herunterladen">
        Vektorgrafik
      </a>
    </li>
  </ul>
  
  <script src="app.js"></script>
</html>
```

## app/app.js anlegen:
```javascript
import $ from 'jquery';

import BpmnModeler from 'bpmn-js/lib/Modeler';

import diagramXML from '../resources/newDiagram.bpmn';

var container = $('#js-drop-zone');

var modeler = new BpmnModeler({
  container: '#js-canvas'
});

function createNewDiagram() {
  openDiagram(diagramXML);
}

async function openDiagram(xml) {
  try {
    await modeler.importXML(xml);

    container.removeClass('with-error').addClass('with-diagram');
  } catch (err) {
    container.removeClass('with-diagram').addClass('with-error');
    container.find('.error pre').text(err.message);
    console.error(err);
  }
}

function registerFileDrop(container, callback) {

  function handleFileSelect(e) {
    e.stopPropagation();
    e.preventDefault();

    var files = e.dataTransfer.files;

    var file = files[0];

    var reader = new FileReader();

    reader.onload = function(e) {

      var xml = e.target.result;

      callback(xml);
    };

    reader.readAsText(file);
  }

  function handleDragOver(e) {
    e.stopPropagation();
    e.preventDefault();

    e.dataTransfer.dropEffect = 'copy'; // Explicitly show this is a copy.
  }

  container.get(0).addEventListener('dragover', handleDragOver, false);
  container.get(0).addEventListener('drop', handleFileSelect, false);
}

// file drag / drop ///////////////////////

// check file api availability
if (!window.FileList || !window.FileReader) {
  window.alert(
    'Looks like you use an older browser that does not support drag and drop. ' +
    'Try using Chrome, Firefox or the Internet Explorer > 10.');
} else {
  registerFileDrop(container, openDiagram);
}

// bootstrap diagram functions

$(function() {

  $('#js-create-diagram').click(function(e) {
    e.stopPropagation();
    e.preventDefault();

    createNewDiagram();
  });

  var downloadLink = $('#js-download-diagram');
  var downloadSvgLink = $('#js-download-svg');

  $('.buttons a').click(function(e) {
    if (!$(this).is('.active')) {
      e.preventDefault();
      e.stopPropagation();
    }
  });

  function setEncoded(link, name, data) {
    var encodedData = encodeURIComponent(data);

    if (data) {
      link.addClass('active').attr({
        'href': 'data:application/bpmn20-xml;charset=UTF-8,' + encodedData,
        'download': name
      });
    } else {
      link.removeClass('active');
    }
  }

  var exportArtifacts = debounce(async function() {

    try {
      const { svg } = await modeler.saveSVG();

      setEncoded(downloadSvgLink, 'diagram.svg', svg);
    } catch (error) {
      console.error('Fehler beim SVG speichern: ', error);
      setEncoded(downloadSvgLink, 'diagram.svg', null);
    }

    try {
      const { xml } = await modeler.saveXML({ format: true });
      setEncoded(downloadLink, 'diagram.bpmn', xml);
    } catch (err) {
      console.error('Fehler beim BPMN speichern: ', err);
      setEncoded(downloadLink, 'diagram.bpmn', null);
    }
  }, 500);

  modeler.on('commandStack.changed', exportArtifacts);
});

function debounce(fn, timeout) {
  var timer;

  return function() {
    if (timer) {
      clearTimeout(timer);
    }

    timer = setTimeout(fn, timeout);
  };
}
```

## CascadingStyleSheets in app/css/app.css anlegen:
```css
* {
  box-sizing: border-box;
}

body, html {

  font-family: "Helvetica Neue",Helvetica, Arial, sans-serif;

  font-size: 12px;

  height: 100%;
  padding: 0;
  margin: 0;
}

a:link {
  text-decoration: none;
}

.content,
.content > div {
  width: 100%;
  height: 100%;
  overflow: hidden;
}

.content > .message {
  text-align: center;
  display: table;

  font-size: 16px;
  color: #111;
}

.content > .message .note {
  vertical-align: middle;
  text-align: center;
  display: table-cell;
}

.content .error .details {
  max-width: 500px;
  font-size: 12px;
  margin: 20px auto;
  text-align: left;
}

.content .error pre {
  border: solid 1px #CCC;
  background: #EEE;
  padding: 10px;
}

.content:not(.with-error) .error,
.content.with-error .intro,
.content.with-diagram .intro {
  display: none;
}


.content .canvas,
.content.with-error .canvas {
  visibility: hidden;
}

.content.with-diagram .canvas {
  visibility: visible;
}

.buttons {
  position: fixed;
  bottom: 20px;
  left: 20px;

  padding: 0;
  margin: 0;
  list-style: none;
}

.buttons > li {
  display: inline-block;
  margin-right: 10px;
}
.buttons > li > a {
  background: #DDD;
  border: solid 1px #666;
  display: inline-block;
  padding: 5px;
}

.buttons a {
  opacity: 0.3;
}

.buttons a.active {
  opacity: 1.0;
}
```

## Initiales Diagramm in resources/newDiagram.bpmn anlegen:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<bpmn2:definitions xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:bpmn2="http://www.omg.org/spec/BPMN/20100524/MODEL" xmlns:bpmndi="http://www.omg.org/spec/BPMN/20100524/DI" xmlns:dc="http://www.omg.org/spec/DD/20100524/DC" xmlns:di="http://www.omg.org/spec/DD/20100524/DI" xsi:schemaLocation="http://www.omg.org/spec/BPMN/20100524/MODEL BPMN20.xsd" id="sample-diagram" targetNamespace="http://bpmn.io/schema/bpmn">
  <bpmn2:process id="Process_1" isExecutable="true">
    <bpmn2:startEvent id="StartEvent_1"/>
  </bpmn2:process>
  <bpmndi:BPMNDiagram id="BPMNDiagram_1">
    <bpmndi:BPMNPlane id="BPMNPlane_1" bpmnElement="Process_1">
      <bpmndi:BPMNShape id="_BPMNShape_StartEvent_2" bpmnElement="StartEvent_1">
        <dc:Bounds height="36.0" width="36.0" x="142" y="122"/>
      </bpmndi:BPMNShape>
    </bpmndi:BPMNPlane>
  </bpmndi:BPMNDiagram>
</bpmn2:definitions>
```

## Projekt bauen
## Produktionsfertig bauen: package.json script einfügen
```json
"all": "run-s lint build",
"build": "webpack --mode production"
```
```
npm run all
```

## Veröffentlichen:
```
scp -r * hostanme-serving-nginx:/var/www/html/npm-example-01
``` 

