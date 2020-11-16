<template>
  <div id="editor" class="diagram">
    <v-btn v-if="!editable" @click="editable = true">Edit</v-btn>
    <span v-else>
      <v-btn @click="openModal">New Node</v-btn>
      <v-btn @click="endEdit">End</v-btn>
    </span>
    <v-btn @click="openInputModal">Import/Export</v-btn>
    <v-btn @click="downloadSVG">Download SVG</v-btn>
    <v-btn @click="isAskClearDiagram = true">Clear Diagram</v-btn>
    <v-btn @click="openSettingsModal">Settings</v-btn>
    <v-navigation-drawer
      v-model="isAskClearDiagram"
      absolute
      temporary
							  right
    > 
      <v-card-text>
		Do you wanna clear the Diagram?
		  </v-card-text>
		<v-list-item color="red" @click="clearDiagram()">
        <v-list-item-content>
          <v-list-item-title class="title">
            Yes
          </v-list-item-title>
        </v-list-item-content>
      </v-list-item >
		<v-list-item color="green" @click="isAskClearDiagram = false">
        <v-list-item-content>
          <v-list-item-title class="title">
            No
          </v-list-item-title>
        </v-list-item-content>
      </v-list-item>
    </v-navigation-drawer>
	  
	  <v-navigation-drawer
      v-model="isModalActive"
      absolute
      temporary
							  right
    > 
    <EditNodeModal
      :node="{ content: {} }"
      @ok="addNode"
      @cancel="cancel"
    />
		  </v-navigation-drawer>
	  
	  <v-navigation-drawer
      v-model="isEditModalActive"
      absolute
      temporary
							  right
    > 
    <EditNodeModal
      :node="tmpNode"
      @ok="editNode"
      @cancel="cancel"
    />
		  </v-navigation-drawer>
	  
	  <v-navigation-drawer
      v-model="isEditLinkModalActive"
      absolute
      temporary
							  right
    > 
    <EditLinkModal
      :link="tmpLink"
      @ok="editLink"
      @cancel="cancel"
    />
		  </v-navigation-drawer>
	  
	  <v-navigation-drawer
      v-model="isInputModalActive"
      absolute
      temporary
							  right
    > 
    <InputModal
      :text="json"
      @ok="importData"
      @cancel="cancel"
    />
		  </v-navigation-drawer>
	  
    <v-navigation-drawer
      v-model="isSettingsModalActive"
      absolute
      temporary
							  right
    >
		<v-card>
			<v-card-text>
				<SettingsModal :settings="settings" v-on:ok="updateSettings" v-on:cancel="isSettingsModalActive = false"/>
			</v-card-text>
		</v-card>
		
		</v-navigation-drawer>
	  
	  <v-layout wrap>
	<v-flex xs2 style="height: calc(100vh - 200px); overflow: auto">	
	<v-list-item v-if="can_edit" @click="editable = !editable">
		  <v-list-item-icon>
            <v-icon>mdi-pencil</v-icon>
          </v-list-item-icon>
        <v-list-item-content>
          <v-list-item-subtitle>
            {{editable ? 'End Edit' : 'Edit'}}
          </v-list-item-subtitle>
        </v-list-item-content>
      </v-list-item>
				<v-divider />
		
		<v-expansion-panels multiple v-model="panel">
			<v-expansion-panel v-if="editable">
				
      <v-expansion-panel-header disable-icon-rotate>
		  Edit Item
				<template v-slot:actions>
            <v-icon color="info">mdi-pencil</v-icon>
          </template>
				</v-expansion-panel-header>
      <v-expansion-panel-content v-if="tmpNode">
		  <!--
 this.tmpNode.id = item.id;
      this.tmpNode.text = item.text;
      this.tmpNode.url = item.url;
      this.tmpNode.color = item.color;
      this.tmpNode.shape = item.shape;
      this.tmpNode.stroke = item.stroke;
      this.tmpNode.strokeWeight = item.strokeWeight;
      this.tmpNode.width = item.width;
      this.tmpNode.height = item.height;
      this.isModalActive = false;
-->
        <v-text-field
            label="Text"
					  hide-details
					  v-if="tmpNode && tmpNode.text"
					  v-model="tmpNode.text"
					  @change="editNode(tmpNode)"
						v-on:keyup="editNode(tmpNode)"
          ></v-text-field>
		  <v-text-field
            label="Text Size"
						type="number"
					  hide-details
						v-if="tmpNode && tmpNode.font_size"
					  v-model="tmpNode.font_size"
						@change="editNode(tmpNode)"
						v-on:keyup="editNode(tmpNode)"
          ></v-text-field>
		  <v-menu :close-on-content-click="false" v-if="tmpNode && tmpNode.font_color">
                      <template v-slot:activator="{ on }">
						  <!--
                        <v-text-field  v-model="tmpNode.font_color.hexa" v-on="on" label="Text Color" @change="editNode(tmpNode)"
						v-on:keyup="editNode(tmpNode)"></v-text-field>
-->
						   <v-btn x-small v-on="on">Text Color</v-btn>
                      </template>
                      <v-color-picker   mode="hexa" v-model="tmpNode.font_color"  hide-mode-switch ></v-color-picker>
                    </v-menu>
		  <v-text-field
            label="URL"
					  hide-details
					  v-model="tmpNode.url"
						@change="editNode(tmpNode)"
						v-on:keyup="editNode(tmpNode)"
          ></v-text-field>
		  
		  <v-select v-if="tmpNode && tmpNode.shape && tmpNode.arrow" label="Line Shape" :items="[{text: 'Bezier Curve', val: 'bezier'},{text: 'Straight Line', val: 'straight'}]" item-text="text" item-value="val"  v-model="tmpNode.shape" placeholder="Select line shape" @change="editLink(tmpNode)"></v-select>
        <v-select label="Pattern" v-if="tmpNode && tmpNode.pattern" v-model="tmpNode.pattern" @change="editLink(tmpNode)" :items="[{text: 'Solid', val: 'solid'},{text: 'Dash', val: 'dash'},{text: 'Dot', val: 'dot'}]" item-text="text" item-value="val" placeholder="Select line pattern" ></v-select>
        <v-select label="Arrow Type" v-if="tmpNode && tmpNode.arrow" v-model="tmpNode.arrow" placeholder="Select arrow type" :items="[{text: 'None', val: 'none'},{text: 'One Side (Source)', val: 'src'},{text: 'One Side (Destination)', val: 'dest'},{text: 'Both', val: 'both'}]" item-text="text" item-value="val" @change="editLink(tmpNode)">
          </v-select>
		  
		  <v-menu :close-on-content-click="false" v-if="tmpNode && tmpNode.color">
                      <template v-slot:activator="{ on }">
						  <!--
                        <v-text-field v-model="tmpNode.color.hexa" v-on="on" label="Color" @change="editNode(tmpNode)"
						v-on:keyup="editNode(tmpNode)"></v-text-field>
-->
						  <v-btn x-small v-on="on">Item Color</v-btn>
                      </template>
                      <v-color-picker  mode="hexa" v-model="tmpNode.color"  hide-mode-switch @change="tmpNode.arrow ? editLink(tmpNode) : editNode(tmpNode)"
						v-on:keyup="tmpNode.arrow ? editLink(tmpNode) : editNode(tmpNode)"></v-color-picker>
                    </v-menu>
		  <v-text-field
            label="Stroke"
						v-if="tmpNode && tmpNode.stroke"
						type="number"
					  hide-details
					  v-model="tmpNode.stroke"
						@change="editNode(tmpNode)"
						v-on:keyup="editNode(tmpNode)"
          ></v-text-field>
		  <v-text-field
            label="Stroke Weight"
						v-if="tmpNode && tmpNode.strokeWeight"
						type="number"
					  hide-details
					  v-model="tmpNode.strokeWeight"
						@change="editNode(tmpNode)"
						v-on:keyup="editNode(tmpNode)"
          ></v-text-field>
		  <v-text-field
            label="Width"
						v-if="tmpNode && tmpNode.width"
						type="number"
					  hide-details
					  v-model="tmpNode.width"
						@change="editNode(tmpNode)"
						v-on:keyup="editNode(tmpNode)"
          ></v-text-field>
		  <v-text-field
            label="Height"
						v-if="tmpNode && tmpNode.height"
						type="number"
					  hide-details
					  v-model="tmpNode.height"
						@change="editNode(tmpNode)"
						v-on:keyup="editNode(tmpNode)"
          ></v-text-field>
      </v-expansion-panel-content>
    </v-expansion-panel>
    <v-expansion-panel v-if="editable">
      <v-expansion-panel-header disable-icon-rotate>
		  Add
		  <template v-slot:actions>
            <v-icon color="info">mdi-plus</v-icon>
          </template>
		</v-expansion-panel-header>
      <v-expansion-panel-content>
        <v-btn icon v-for="(item,index) in diagram_items" @click="diagramAdd(item.key)" :key="'shape-'+index"><v-icon>{{item.icon}}</v-icon></v-btn>
      </v-expansion-panel-content>
    </v-expansion-panel>
			
		<v-expansion-panel>
      <v-expansion-panel-header disable-icon-rotate>
		  Settings
			<template v-slot:actions>
            <v-icon color="info">mdi-cog</v-icon>
          </template>
			</v-expansion-panel-header>
      <v-expansion-panel-content>
         <v-list
        dense
        nav
      >
        <v-list-item @click="toggleGrid()">
          <v-list-item-icon>
            <v-icon>mdi-grid</v-icon>
          </v-list-item-icon>

          <v-list-item-content>
            <v-list-item-title>Toggle Grid</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
		<v-list-item @click="isAskClearDiagram = true">
          <v-list-item-icon>
            <v-icon>mdi-nuke</v-icon>
          </v-list-item-icon>

          <v-list-item-content>
            <v-list-item-title>Clear Diagram</v-list-item-title>
          </v-list-item-content>
        </v-list-item>	
				 <v-list-item @click="openInputModal">
          <v-list-item-icon>
            <v-icon>mdi-swap-vertical</v-icon>
          </v-list-item-icon>

          <v-list-item-content>
            <v-list-item-title>Import/Export</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
				 <v-list-item @click="downloadSVG">
          <v-list-item-icon>
            <v-icon>mdi-download</v-icon>
          </v-list-item-icon>

          <v-list-item-content>
            <v-list-item-title>Download SVG</v-list-item-title>
          </v-list-item-content>
        </v-list-item>	
				 <v-list-item @click="save()">
          <v-list-item-icon>
            <v-icon>mdi-content-save</v-icon>
          </v-list-item-icon>

          <v-list-item-content>
            <v-list-item-title>Save</v-list-item-title>
          </v-list-item-content>
        </v-list-item>	
      </v-list>
      </v-expansion-panel-content>
    </v-expansion-panel>
  </v-expansion-panels>
		
			
   
			
	</v-flex>
		 <v-flex xs10 style="max-height: calc(100vh - 200px);max-width: calc(100vw - 500);overflow: auto">
    <Diagram
      :width="graphData.width ? graphData.width : 800"
      :height="graphData.height ? graphData.height : 600"
      :fluid="settings.isFluid"
      :scale="settings.scale"
      :background="graphData.background || '#fafafa'"
      :showGrid="graphData.showGrid"
      :nodes="graphData.nodes"
      :links="graphData.links"
      :editable="editable"
      :labels="
        graphData.labels || {
          edit: 'Edit',
          remove: 'Remove',
          link: 'New Link',
          select: 'Select',
          cancel: 'Cancel',
          copy: 'Copy'
        }
      "
      @editNode="openNodeEdit"
      @editLink="openLinkEdit"
      @nodeClicked="nodeClicked"
      @linkClicked="linkClicked"
      @nodeChanged="nodeChanged"
      @linkChanged="linkChanged"
    >
    </Diagram>
			 			 
	</v-flex>
	
		 </v-layout>
  </div>
</template>

<script>

	
import Diagram from "./Diagram";
import EditNodeModal from "@/lib/EditNodeModal";
import EditLinkModal from "@/lib/EditLinkModal";
import InputModal from "@/lib/InputModal";
import AskModal from "@/lib/AskModal";
import SettingsModal from "@/lib/SettingsModal";
export default {
  name: "DiagramEditor",
  components: {
    Diagram,
    EditNodeModal,
    EditLinkModal,
    InputModal,
    AskModal,
    SettingsModal
  },
  props: {
    value: {
      type: Object,
      default: () => {
        return {
          width: 2000,
          height: 1000,
          background: "#fafafa",
          showGrid: false,
          labels: {
            edit: "Edit",
            remove: "Remove",
            link: "New Link",
            select: "Select",
            copy: "Copy"
          },
          nodes: [],
          links: []
        };
      }
    },
	  height: null,
	  width: null,
	  edit_mode: false,
	  can_edit: true
  },
	watch: {
		value(val){
			this.init()
		},
		'tmpNode'(val){
			console.log('color changed')
			this.editNode(this.tmpNode)
		},
		'tmpNode.font_color'(val){
			console.log('font_color changed')
			if(!this.tmpNode.arrow){
				this.editNode(this.tmpNode)
			}
			
		},
	},
  computed: {
    graphData: {
      get() {
        return this.value;
      },
      set(val) {
        this.$emit("input", val);
      }
    }
  },
  data() {
    return {
		panel: [2],
		diagram_items: [
				{title: 'Square',icon: 'mdi-shape-square-plus',key: 'square'},
				{title: 'Rectangle',icon: 'mdi-shape-rectangle-plus',key: 'rectangle'},
				{title: 'Circle',icon: 'mdi-shape-circle-plus',key: 'circle'},
				{title: 'Elipse',icon: 'mdi-shape-oval-plus',key: 'elipse'},
				{title: 'Text',icon: 'mdi-text',key: 'text'},
			],
      name: "",
      url: "",
      color: "",
      json: "",
      isModalActive: false,
      isEditModalActive: false,
      isEditLinkModalActive: false,
      isInputModalActive: false,
      isSettingsModalActive: false,
      editable: false,
      settings: {
        width: 1500,
        height: 2500,
        isFluid: false,
        scale: "1",
        showGrid: false
      },
      tmpNode: {
        id: "",
        shape: "rectangle",
        width: 0,
        height: 0,
		  text: "",
          url: "",
          color: "",
        font_color: '',
		  font_size: ''
      },
      tmpLink: {
        id: "",
        content: {
          color: "",
          pattern: "solid",
          arrow: "none"
        }
      },
      isAskClearDiagram: false
    };
  },
  methods: {
	  init(){
		  /*
			let diagram_width = window.innerWidth - 325
			let diagram_height = window.innerHeight - 255
			if(this.height){
				this.graphData.height = this.height
				this.settings.height = this.height
			}else{
				this.graphData.height = diagram_height
				this.settings.height = diagram_height
			}
		  if(this.width){
				this.graphData.width = this.width
			  this.settings.width = this.width
			}else{
				this.graphData.width = diagram_width
				this.settings.width = diagram_width
			}
			*/
		},
	  save(){
		this.$emit('update',this.graphData)  
	  },
	   diagramAdd(item){
		   let default_item_color={"alpha":1,"hex":"#ECF0F1","hexa":"#ECF0F1FF","hsla":{"h":192,"s":0.151515151515151,"l":0.9352941176470588,"a":1},"hsva":{"h":192,"s":0.020746887966804906,"v":0.9450980392156862,"a":1},"hue":192,"rgba":{"r":236,"g":240,"b":241,"a":1}}
let default_font_color={"alpha":1,"hex":"#34495E","hexa":"#34495EFF","hsla":{"h":210,"s":0.2876712328767125,"l":0.28627450980392155,"a":1},"hsva":{"h":210,"s":0.4468085106382979,"v":0.3686274509803922,"a":1},"hue":210,"rgba":{"r":52,"g":73,"b":94,"a":1}}
		   
		  switch(item){
			  case 'square':
				  this.addNode(
					 { "text": "New Square", "width": 60, "height": 60, "shape": "rectangle", "stroke": 0, strokeWeight: 0, "point": { "x": 50,
			"y": 66.4611318108179 }, font_color: default_font_color, color: default_item_color, font_size: 14 } 
					) 
					  break;
					  case 'rectangle':
					  this.addNode(
					 {  "text": "New Rectangle" , "width": 100, "height": 60, "shape": "rectangle", "stroke": 0, strokeWeight: 0, "point": { "x": 50,
			"y": 66.4611318108179 }, font_color: default_font_color, font_size: 14, color: default_item_color } 
					) 
				  break;
				  case 'circle':
					  this.addNode(
					 {  "text": "New Circle" , "width": 60, "height": 60, "shape": "ellipse", "stroke": 0, strokeWeight: 0, "point": { "x": 50,
			"y": 66.4611318108179 }, font_color: default_font_color, font_size: 14, color: default_item_color } 
					) 
				  break;
				  case 'elipse':
					  this.addNode(
					 {   "text": "New Elipse" , "width": 100, "height": 60, "shape": "ellipse", "stroke": 0, strokeWeight: 0, "point": { "x": 50,
			"y": 66.4611318108179 }, font_color: default_font_color, font_size: 14, color: default_item_color } 
					) 
				  break;
				  case 'text':
					  this.addNode(
					 { font_color: {hexa: '#34495e'}, color: {hexa: '#ecf0f1'},  "text": "New Text", "width": 100, "height": 35, "shape": "text", "stroke": 0, strokeWeight: 0, "point": { "x": 9.999999999999993, "y": 34.31059443007615 } } 
					) 
				  break;
		  }
		 
	  },
    clearDiagram() {
      this.graphData.nodes = [];
      this.graphData.links = [];
      this.isAskClearDiagram = false;
    },
    generateID() {
      return (
        new Date().getTime().toString(16) +
        Math.floor(Math.random() * 1000000).toString(16)
      );
    },
    openModal() {
      this.isModalActive = true;
    },
    cancel() {
      this.isModalActive = false;
      this.isEditModalActive = false;
      this.isEditLinkModalActive = false;
      this.isInputModalActive = false;
      this.isAskClearDiagram = false;
      this.isSettingsModalActive = false;
    },
    addNode(item) {
      this.graphData.nodes.push({
        id: this.generateID(),
		  text: item.text,
          url: item.url,
          color: item.color,
			font_color: item.font_color,
			font_size: item.font_size,
        width: parseInt(item.width) || 150,
        height: parseInt(item.height) || 60,
        stroke: item.stroke,
        strokeWeight: item.strokeWeight,
        shape: item.shape,
        point: {
          x: 10,
          y: 100 + Math.random() * 100
        }
      });
      this.isModalActive = false;
    },
    openNodeEdit(item) {
		this.tmpNode = item
	/*
      this.tmpNode.id = item.id;
      this.tmpNode.text = item.text;
      this.tmpNode.url = item.url;
      this.tmpNode.color = item.color;
		this.tmpNode.font_color = item.font_color;
		this.tmpNode.font_size = item.font_size;
      this.tmpNode.shape = item.shape;
      this.tmpNode.stroke = item.stroke;
      this.tmpNode.strokeWeight = item.strokeWeight;
      this.tmpNode.width = item.width;
      this.tmpNode.height = item.height;
	  */
      //this.isModalActive = false;

      //this.isEditModalActive = true;
		if(!this.panel.includes(1)){
			this.panel.push(1)
		}
		
    },
    editNode(item) {
      let tmp = this.graphData.nodes.find(x => x.id === item.id);
	tmp.id = item.id;
      tmp.text = item.text;
      tmp.url = item.url;
      tmp.color = item.color;
		tmp.font_color = item.font_color;
		tmp.font_size = item.font_size;
      tmp.shape = item.shape;
      tmp.stroke = item.stroke;
      tmp.strokeWeight = item.strokeWeight;
      tmp.width = parseInt(item.width);
      tmp.height = parseInt(item.height);
     // this.isEditModalActive = false;
	
    },
    openLinkEdit(item) {
      //this.tmpLink.id = item.id;
     // this.tmpLink.content = Object.assign({}, item.content);
     // this.isEditLinkModalActive = true;
		this.tmpNode = item
		this.tmpNode.id = item.id;
		//this.tmpNode.content = Object.assign({}, item.content);
		
		item.color ? this.tmpNode.color = item.color : this.tmpNode.color = {"alpha":1,"hex":"#ECF0F1","hexa":"#ECF0F1FF","hsla":{"h":192,"s":0.151515151515151,"l":0.9352941176470588,"a":1},"hsva":{"h":192,"s":0.020746887966804906,"v":0.9450980392156862,"a":1},"hue":192,"rgba":{"r":236,"g":240,"b":241,"a":1}}
		
		item.arrow ? this.tmpNode.arrow = item.arrow : this.tmpNode.arrow = 'none'
		item.pattern ? this.tmpNode.pattern = item.pattern : this.tmpNode.pattern = 'solid'
		item.shape ? this.tmpNode.shape = item.shape : this.tmpNode.shape = 'straight'
    },
    editLink(item) {
      let tmp = this.graphData.links.find(x => x.id === item.id);
      tmp.color = item.color;
      tmp.shape = item.shape;
      tmp.pattern = item.pattern;
      tmp.arrow = item.arrow;
      //this.isEditLinkModalActive = false;
    },
    endEdit() {
      this.editable = false;
    },
    nodeClicked(id) {
      this.$emit("nodeClicked", id);
		console.log("nodeClicked", id)
    },
    linkClicked(id) {
      this.$emit("linkClicked", id);
		console.log("linkClicked", id)
    },
    nodeChanged(obj) {
		console.log('nodeChanged',obj)
      this.graphData.nodes = obj.nodes;
    },
    linkChanged(obj) {
		console.log('linkChanged',obj)
      this.graphData.links = obj.links;
    },
    openInputModal() {
      this.isInputModalActive = true;
      this.json = JSON.stringify(this.graphData);
    },
    importData(value) {
      const obj = JSON.parse(value.text);
      if (obj) {
        this.graphData = obj;
        this.isInputModalActive = false;
      }
    },
    downloadSVG() {
      const blob = new Blob(
        [document.getElementById("svg-diagram-show-area").innerHTML],
        {
          type: "image/svg+xml"
        }
      );
      const url = window.URL.createObjectURL(blob);
      const link = document.createElement("a");
      link.href = url;
      link.download = "image.svg";
      link.click();
    },
    changeGrid() {
      this.graphData.width = parseInt(this.settings.width);
      this.graphData.height = parseInt(this.settings.height);
      this.graphData.showGrid = this.settings.showGrid;
    },
	  toggleGrid(){
		  this.graphData.showGrid = !this.graphData.showGrid
	  },
    openSettingsModal() {
      this.isSettingsModalActive = true;
      this.settings.width = this.graphData.width;
      this.settings.height = this.graphData.height;
      this.settings.showGrid = this.graphData.showGrid;
		this.updateSettings(this.settings)
    },
    updateSettings(val) {
      this.settings = Object.assign({}, val);
      this.changeGrid();
     // this.isSettingsModalActive = false;
    }
  }
};
</script>
