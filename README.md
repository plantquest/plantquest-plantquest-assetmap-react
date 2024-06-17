# @plantquest/assetmap-react

[![npm version](https://img.shields.io/npm/v/@plantquest/assetmap-react.svg)](https://www.npmjs.com/package/@plantquest/assetmap-react)

![Logo_Plantquest_horizontal](https://user-images.githubusercontent.com/89012461/223706137-a9682633-cebb-47ab-8bdf-0452e6b7fff4.png)
>PlantQuest is a supplier of asset mapping solutions and technology to the Life Science sector. The industry focused PlantQuest Asset Map allows users to intergrate the PlantQuest mapping technology into their technology stack. Upon doing so, users can manipulate maps and display asset information within the context of where it is generated helping workers to make smarter, quicker and more insightful decisions. In turn, increasing their operational effeciency.

## Install

```bash
npm install --save @plantquest/assetmap-react
```

## Usage



## Debug Log

Set `window.PLANTQUEST_ASSETMAP_LOG` to `true` to enable logging.

Set `window.PLANTQUEST_ASSETMAP_DEBUG.show_coords` to `true` to display a small box where asset info is shown, and xco-yco of cursor when clicked on the map.


## Options

* `width`: Pixel width of map ( default: `'600px'` )
* `height`: Pixel height of map ( default: `'400px'` )
* `mapImg`: Map dimensions - very important for the polygons to fit the map 
* `mapBounds`: Pixel bounds of map
* `mapStart`: Pixel start position of map ( e.g `[y, x]` ( default: `[2925, 3900]` ) )
* `mapStartZoom`: Starting zoom level
* `mapRoomFocusZoom`: Zoom level for room focus
* `mapMaxZoom`: Maximum zoom
* `mapMinZoom`: Minimum zoom
* `states`: State definitions ( optional )
  * `{ [stateName]: { color: COLOR, name: STRING, marker: 'standard'|'alert'}, ...}`
* `start.map`: Starting map ( default: `0` )
* `start.level`: Starting level ( default: `0` )
* `room.color`: Room highlight color ( default: `'#33f'` )
* `mode`: MAP MODE can either can `'live'` or `'demo'` - check out the code example below for details
* `apikey`: Your project api key
* `tilesEndPoint`: Endpoint to your maps ( note that use tilesets so we don't specify maps explicitly )
* `plant_id`: Your plant ID
* `project_id`: Your project ID
* `stage`: Your stage

## ReactJS: Quick Example

```js

import { PlantQuestAssetMap } from '@plantquest/assetmap-react'

// enable logging - useful for debugging purposes
window.PLANTQUEST_ASSETMAP_LOG = true
// enable small info box for the current asset info shown, or for the position of your mouse on the map - such as xco, yco, etc.
window.PLANTQUEST_ASSETMAP_DEBUG.show_coords = true

const options = {
  data: 'https://demo.plantquest.app/sample-data.js', // not needed if using: `mode: 'live'`

  width: '100%',
  height: '100%',
  // this will enable dynamic resizing of the map widget
  // it will adjust to your node ( 100% will take 100% of your parent node, etc. )
  // but then the parent node of the component has to have its own width and height
  // like in the example below
  
  mapImg: [6140, 4602], // important: set the map [width, height]
  
  states: {
    up: { color: '#696', name: 'Up', marker: 'standard' },
    down: { color: '#666', name: 'Down', marker: 'standard' },
    missing: { color: '#f3f', name: 'Missing', marker: 'alert' },
    alarm: { color: '#f33', name: 'Alarm', marker: 'alert' },
    // "color" - color of the polygon of that state
    // "name" - name of the state
    // "marker" - type of marker ( 'standard' | 'alert' ) 
  },
  
  endpoint: ENDPOINT, // your endpoint: 'https://*'
  apikey: '<STRING>', // your api key
  tilesEndPoint: '<STRING>', // map tiles endpoint

  // mode can either can 'live' or 'demo'
  // if you want data to be loaded from the static demo js file (self.data here) - use 'demo' mode
  // if you want 'live' data from the endpoint - use 'live' mode
  mode: 'live',

  plant_id: '<STRING>', // your plant_id
  project_id: '<STRING>', // your project_id
  stage: '<STRING>', // your stage
  
  // room highlight color
  room: {
    color: 'red'
  },
  
}

// container when showing an asset
/*
// css example
div.plantquest-assetmap-asset-state-up {
    color: white;
    border: 2px solid #696;
    border-radius: 4px;
    background-color: #696;
    opacity: 0.7;
}
*/
class AssetInfo extends React.Component {
  constructor(props) {
    super(props)
    this.state = {}
  }
  
  render() {
    return <div>
         <h3>{this.props.asset.tag}</h3>
         <p><i>Building:</i> {this.props.asset.building}</p>
       </div>
  }
}

class App extends React.Component {
  
  constructor(props) {
    super(props)
    
    // to keep track of map's state
    // using listeners so we can reuse these in our app
    this.state = {
      map: -1,
      level: '',
      rooms: [],
      showRoom: null,
      showAsset: null,
    }
    

  }
  
  componentDidMount() {
    console.log("Mounting..")

    const PQAM = window.PlantQuestAssetMap
    
    // set up message listener
    PQAM.listen((msg) => {
      // put 'ready' listener to use
      if('ready' === msg.state) {
        // set 'rooms' for reuse
        this.setState({
          rooms: PQAM.data.rooms
        })
      }
      // when a user selects a room
      // "USER SELECT ROOM" example
      else if ('room' === msg.select) {
        // pick a room
        let item = PQAM.data.roomMap[msg.room]
        this.setState({ showRoom: item })
        this.selectRoom(item)
      }
      // "USER SELECT MAP" example
      else if('map' === msg.show) {
        this.setState({ level: msg.level })
        this.setState({ map: msg.map })
      }
      // Listen for "USER SHOW ASSET"
      else if('asset' === msg.show) {
        // use msg.asset
         this.showAsset(asset)
      }
      
    })
  
  }
  
  selectRoom(item) {
    const PQAM = window.PlantQuestAssetMap
    
    // "SEND A MESSAGE" example
    // "SHOW ROOM"
    PQAM.send({
      srv: 'plantquest',
      part: 'assetmap',
      show: 'room',
      room: item.room,
      focus: true,
    })
  
  }
  
  showAsset(asset) {
    const PQAM = window.PlantQuestAssetMap
    
    // "SHOW ASSET" example
    // when showing an asset
    // it's important to first show the room of that asset and then the asset
    PQAM.send({
      srv: 'plantquest',
      part: 'assetmap',
      show: 'room',
      room: asset.room,
      focus: true,
    })
    PQAM.send({
      srv: 'plantquest',
      part: 'assetmap',
      show: 'asset',
      asset: asset.id,
    })
    
    this.setState({ showRoom: asset.room })
    this.setState({ showAsset: asset })
    
  }
  

  render() {
    return (
      <div className="App">
        <div style={{width: '150vh', height: '100vh' }}>
          <PlantQuestAssetMap
            options={options}
            assetinfo={AssetInfo}
          />
        </div>
      </div>
    )
  }
  
}

```

## Messages

  <h2>SEND MESSAGES</h2>

  <h3>ZOOM</h3>
  <pre>
{
  srv: 'plantquest',
  part: 'assetmap',
  zoom: &lt;INTEGER&gt;,     
}</pre>
  <p>Where:<br>
  <span>When you zoom on a map, the scale of the map changes to showing more or less of your facility map, depending on whether you zoom in or out. The default zoom level value is between 2 and 6, with this value having the ability to be adjusted as needed.</span><br>
    <i>&lt;INTEGER&gt;</i>: Zoom level (default: 2 to 6)<br>
  </p>


  <h3>GET RELATION ( REQUIRES A RELATION LISTENER )</h3>
  <pre>
{
  srv: 'plantquest',
  part: 'assetmap',
  relate: 'room-asset',
}</pre>
<p>Where:<br>
  <span>This retrieves information and relationships regarding a room and the assets within it.</span><br>
  <p>Listen: <a href="#listen-relation">RELATION</a><br>
  </p>


  <h3>SHOW ROOM</h3>
  <pre>
{
  srv: 'plantquest',
  part: 'assetmap',
  show: 'room',
  room: &lt;ROOM-ID&gt;,
  focus: &lt;Boolean&gt;,   
}</pre>
  <p>Where:<br>
    <span>This is used to display information regarding a particular room on the asset map. When a room is selected on the map, the asset map will focus on the room.</span><br>
    <i>&lt;ROOM-ID&gt;</i>: Room Identifier String<br>
    <i>&lt;Boolean&gt;</i>: either true or false - enable focus when a room is shown<br>
  </p>

  <h3>SHOW ASSET</h3>
  <pre>
{
  srv: 'plantquest',
  part: 'assetmap',
  show: 'asset',
  asset: &lt;ASSET-ID&gt;,     
}</pre>
  <p>Where:<br>
  <span>This calls the PlantQuest asset map via an API endpoint and displays a particular asset location and asset ID on the map.</span><br>
    <i>&lt;ASSET-ID&gt;</i>: Asset Identifier String<br>
  </p>
  
  <h3>HIDE ASSET</h3>
  <pre>
{
  srv: 'plantquest',
  part: 'assetmap',
  hide: 'asset',
  asset: &lt;ASSET-ID&gt;,     
}</pre>
  <p>Where:<br>
  <span>This calls the PlantQuest asset map via an API endpoint and removes a particular asset from the map.</span><br>
    <i>&lt;ASSET-ID&gt;</i>: Asset Identifier String<br>
  </p>

  <h3>SET ASSET STATE</h3>
  <pre>
{
  srv: 'plantquest',
  part: 'assetmap',
  show: 'asset',
  state: &lt;STATE&gt;,
  asset: &lt;ASSET-ID&gt;,     
}</pre>
  <p>Where:<br>
  <span>This indicates what asset information to show on the asset map, and in what manner to display it.</span><br>
    <i>&lt;STATE&gt;</i>: State String ('up', 'down', 'alarm', 'missing') - states from the <b>options</b> <br>
    <i>&lt;ASSET-ID&gt;</i>: Asset Identifier String<br>
  </p>

  <h3>SHOW MAP</h3>
  <pre>
{
  srv: 'plantquest',
  part: 'assetmap',
  show: 'map',
  map: &lt;INTEGER&gt;,     
}</pre>
  <p>Where:<br>
  <span>This indicates what map to show. Maps are numbered 1 to n. For example, "Map 1" may be "First Floor Map", "Map 2" may be "Second Floor Map" etc.</span><br>
    <i>&lt;INTEGER&gt;</i>: Number of the map<br>
  </p>
 
   <h3>LIST ASSET|ROOM|BUILDING</h3>
  <pre>
{
  srv: 'plantquest',
  part: 'assetmap',
  list: 'asset|room|building', 
}</pre>

```js
// for example - take assets
const PQAM = window.PlantQuestAssetMap
// our listener
PQAM.listen((msg) => {
  if('asset' === msg.list) {
    // use msg.assets
    // where msg.assets is a list of all assets on the map
  }
})

PQAM.send({
  'srv': 'plantquest',
  'part': 'assetmap',
  'list': 'asset'
})

// the syntax is flexiable enough for us to just write:

PQAM.send('srv:plantquest,part:assetmap,list:asset') 

```

<h3>LOAD ASSET|ROOM|BUILDING</h3>
  <pre>
{
  srv: 'plantquest',
  part: 'assetmap',
  load: 'asset|room|building', 
  id: &lt;STRING&gt;,
}</pre>
 <p>Where:<br>
  <span>Load a single <code>asset|room|building</code> by id</span><br>
 <i>&lt;STRING&gt;</i>: ID <code>(UUIDv4 format)</code> of the <code>asset|room|building</code> to be loaded
  </p>

<h3>SAVE ASSET|ROOM|BUILDING</h3>
  <pre>
{
  srv: 'plantquest',
  part: 'assetmap',
  save: 'asset|room|building', 
  asset|room|building: &lt;OBJECT&gt;,
}</pre>
 <p>Where:<br>
  <span>Save a new <code>asset|room|building</code> with your own metadata</span><br>
    <i>&lt;OBJECT&gt;</i>: Metadata of your <code>asset|room|building</code> to be saved
  </p>

```js
// for example
const PQAM = window.PlantQuestAssetMap
    
PQAM.send({
  srv: 'plantquest',
  part: 'assetmap',
  save: 'asset',
  asset: {
    id: 'e565b059-8633-460a-8171-903d38720c26',
    tag: 'asset001',
    xco: 10,
    yco: 10,
  },
})
    
PQAM.listen((msg) => {
  if('asset' === msg.save) {
    // use msg.asset
    // where msg.asset is the newly saved asset
  }
})
   
```
<h3>REMOVE ASSET|ROOM|BUILDING</h3>
  <pre>
{
  srv: 'plantquest',
  part: 'assetmap',
  remove: 'asset|room|building',
  id: &lt;STRING&gt;,
}</pre>
 <p>Where:<br>
  <span>remove <code>asset|room|building</code> by id</span><br>
 <i>&lt;STRING&gt;</i>: ID <code>(UUIDv4 format)</code> of the <code>asset|room|building</code>
  </p>
  
<h3>SHOW LIST OF ASSETS ON THE MAP</h3>
  <pre>
{
  srv: 'plantquest',
  part: 'assetmap',
  show: 'asset',
  state: &lt;STRING&gt;,
  asset: &lt;ARRAY&gt;,
}</pre>
 <p>Where:<br>
  <span>display a list of assets by id</span><br>
 <i>&lt;STRING&gt;</i>: STATE is generally user-specified - for example it can be 'up'|'down'|'alarm'|'missing'
 
 <i>&lt;ARRAY&gt;</i>: IDs <code>(UUIDv4 format)</code> of the <code>assets</code> to be shown on the map using clustering - note: if this is set to <code>null</code>, all assets will be rendered and displayed
  </p>
  
  
  <h2>LISTEN MESSAGES</h2>
   
  <h3>STATE</h3>
  <pre>
{
  srv: 'plantquest',
  part: 'assetmap',
  state: &lt;STATE&gt;,
}</pre>
  <p>Where:<br>
  <span>Provides an explanation of what the "<STATE>" placeholder stands for, indicating that it should be replaced with the actual state string. In this case, the only available state is 'ready'.</span><br>
    <i>&lt;STATE&gt;</i>: 'ready' - triggered when the map is fully rendered <br>
  </p>


  <a name="listen-relation"></a>
  <h3>RELATION</h3>
  <pre>
{
  srv: 'plantquest',
  part: 'assetmap',
  relate: 'room-asset',
  relation: &lt;RELATION&gt;,
}</pre>
  <p>Where:<br>
    <i>&lt;RELATION&gt;</i>:
    <pre>
      { '&lt;ROOM-ID&gt;': { asset: [ '&lt;ASSET-ID&gt;', ... ] } }</pre>
    ROOM-ASSET RELATION: Get all the rooms IDS containing their asset IDS in that room
    <br>
  </p>

```js
// for example
const PQAM = window.PlantQuestAssetMap
PQAM.listen((msg) => {
  if('room-asset' === msg.relate) {
    // use msg.relation
  }
})
```


  <h3>USER SELECT ROOM</h3>
  <pre>
{
  srv: 'plantquest',
  part: 'assetmap',
  select: 'room',
  room: &lt;ROOM-ID&gt;,     
}</pre>
  <p>Where:<br>
  <span>Relates to when the user selects a room on the map and returns the rooms id and name.</span><br>
    <i>&lt;ROOM-ID&gt;</i>: Room Identifier String<br>
  </p>

  <h3>USER SELECT MAP</h3>
  <pre>
{
  srv: 'plantquest',
  part: 'assetmap',
  show: 'map',
  map: &lt;INTEGER&gt;,
  level: &lt;STRING&gt;,
}</pre>
  <p>Where:<br>
  <span>Relates to the map that user wishes to view and manipulate. Example: User is currently viewing "Map 1 - First Floor" and wishes to view "Map 2 - Second Floor".</span><br>
    <i>&lt;INTEGER&gt;</i>: Number of the map user just selected <br>
    <i>&lt;STRING&gt;</i>: Name of the level of that map <br>
  </p>
  
  <h3>USER SHOW ASSET</h3>
  <pre>
{
  srv: 'plantquest',
  part: 'assetmap',
  show: 'asset',
  asset: &lt;OBJECT&gt;,
}</pre>
  <p>Where:<br>
  <span>Allows the user to view an assets location on the asset map.</span><br>
    <i>&lt;OBJECT&gt;</i>: Metadata of the SHOWN asset <br>
  </p>

  <h3>USER LIST ASSET|ROOM|BUILDING</h3>
  <pre>
{
  srv: 'plantquest',
  part: 'assetmap',
  list: 'asset|room|building',
  assets|rooms|buildings: &lt;ARRAY&gt;,
}</pre>
  <p>Where:<br>
 <span>Allows the user to list all the <code>assets|rooms|buildings</code> there are on the map.</span><br>
 <i>&lt;ARRAY&gt;</i>: List of all the <code>assets|rooms|buildings</code> <br>
  </p>

<h3>USER LOAD ASSET|ROOM|BUILDING</h3>
  <pre>
{
  srv: 'plantquest',
  part: 'assetmap',
  load: 'asset|room|building',
  asset: &lt;OBJECT&gt;,
}</pre>
  <p>Where:<br>
  <span>Allows the user to load a single <code>asset|room|building</code> by id</span><br>
 <i>&lt;OBJECT&gt;</i>: the metadata of the loaded <code>asset|room|building</code> <br>
  </p>

<h3>USER SAVE ASSET|ROOM|BUILDING</h3>
  <pre>
{
  srv: 'plantquest',
  part: 'assetmap',
  save: 'asset|room|building',
  asset|room|building: &lt;OBJECT&gt;,
}</pre>
  <p>Where:<br>
  <span>Allows the user to save a new <code>asset|room|building</code> with their own metadata </span><br>
 <i>&lt;OBJECT&gt;</i>: the metadata of the newly saved <code>asset|room|building</code> <br>
  </p>
  
<h3>USER REMOVE ASSET|ROOM|BUILDING</h3>
  <pre>
{
  srv:'plantquest',
  part:'assetmap',
  remove:'asset|room|building',
  asset|room|building: &lt;ID&gt;,
}</pre>
  <p>Where:<br>
  <span>Allows the user to see when an item is removed </span><br>
 <i>&lt;ID&gt;</i>: the id of the removed <code>asset|room|building</code> <br>
  </p> 

  
## Licenses

[MIT](LICENSE) © [Plantquest Ltd](https://plantquest.com)
[BSD 2-Clause](LEAFLET-LICENSE) © [Vladimir Agafonkin, Cloudmade](https://leafletjs.com/)
[MIT](LICENSE) © [Justin Manley](https://github.com/Leaflet/Leaflet.toolbar)
