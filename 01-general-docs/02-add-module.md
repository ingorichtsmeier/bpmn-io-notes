# Modul (hier: Property Panel) hinzufügen

Projekt existiert bereits

## npm Module hinzufügen
```
npm i bpmn-js-properties-panel --save
npm i camunda-bpmn-moddle --save
```

## webpack.config.js anpassen
```
{ from: 'node_modules/bpmn-js-properties-panel/dist/assets', to: 'vendor/bpmn-js-properties-panel/assets' }
```
npm run start

## Properties Panel mit Styles in Html-Seite einfügen:
```html
<link rel="stylesheet" href="vendor/bpmn-js-properties-panel/assets/bpmn-js-properties-panel.css" />

<div class="properties-panel-parent" id="js-properties-panel"></div>
```

## Properties Panel Module in app.js laden und einbinden:
```javascript
import propertiesPanelModule from 'bpmn-js-properties-panel';
import propertiesProviderModule from 'bpmn-js-properties-panel/lib/provider/camunda';
import camundaModdleDescriptor from 'camunda-bpmn-moddle/resources/camunda.json'
```

```javascript
  propertiesPanel: {
    parent: '#js-properties-panel'
  },
  additionalModules: [
    propertiesPanelModule,
    propertiesProviderModule
  ],
  moddleExtensions: {
    camunda: camundaModdleDescriptor
  }
```

## CascadingStyleSheets erweitern
löschen:
```css
.content > div {
```

einfügen:
```css
.content: {
  position: relative;
```

einfügen:
```css
.content > .message {
  width: 100%;
  height: 100%;
```

einfügen:
```css
.content .canvas {
  width: 100%;
}
.content .canvas,
.content .properties-panel-parent {
  display: none;
}
.content.with-diagram .canvas,
.content.with-diagram .properties-panel-parent {
  display: block;
}
```

einfügen:
```css
.properties-panel-parent {
  border-left: 1px solid #ccc;
  overflow: auto;
}
.properties-panel-parent:empty {
  display: none;
}
.properties-panel-parent > .djs-properties-panel {
  padding-bottom: 70px;
  min-height: 100%;
}
```

vollständige, funktionierende Vorlage: [app.css](app.css)

npm run all

