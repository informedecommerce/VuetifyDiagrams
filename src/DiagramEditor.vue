<template>
  <div id="editor">
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
	<v-flex xs2>	
		
		<v-navigation-drawer permanent>
      <v-list-item v-if="can_edit" @click="editable = !editable">
        <v-list-item-content>
          <v-list-item-subtitle>
            Edit
          </v-list-item-subtitle>
        </v-list-item-content>
      </v-list-item>
			<v-list-item >
        <v-list-item-content>
          <v-list-item-title class="title">
            Add
          </v-list-item-title>
          <v-list-item-subtitle>
            Quick add Items
          </v-list-item-subtitle>
        </v-list-item-content>
      </v-list-item>

      <v-divider></v-divider>
<v-btn icon v-for="(item,index) in diagram_items" @click="diagramAdd(item.key)" :key="'shape-'+index"><v-icon>{{item.icon}}</v-icon></v-btn>
     <!--
			<v-list
        dense
        nav
      >
        <v-list-item
          v-for="item in diagram_items"
          :key="item.title"
          link
					 @click="diagramAdd(item.key)"
        >
          <v-list-item-icon>
            <v-icon>{{ item.icon }}</v-icon>
          </v-list-item-icon>

          <v-list-item-content>
            <v-list-item-title>{{ item.title }}</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
      </v-list>
			-->
			<v-divider />
			<v-list-item >
        <v-list-item-content>
          <v-list-item-title class="title">
            Settings
          </v-list-item-title>
          <v-list-item-subtitle>
            Quick Settings
          </v-list-item-subtitle>
        </v-list-item-content>
      </v-list-item>
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
				 <v-list-item @click="save()">
          <v-list-item-icon>
            <v-icon>mdi-content-save</v-icon>
          </v-list-item-icon>

          <v-list-item-content>
            <v-list-item-title>Save</v-list-item-title>
          </v-list-item-content>
        </v-list-item>	
      </v-list>
    </v-navigation-drawer>
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
		}
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
		diagram_items: [
				{title: 'Square',icon: 'mdi-shape-square-plus',key: 'square'},
				{title: 'Rectangle',icon: 'mdi-shape-rectangle-plus',key: 'rectangle'},
				{title: 'Circle',icon: 'mdi-shape-circle-plus',key: 'circle'},
				{title: 'Elipse',icon: 'mdi-shape-oval-plus',key: 'elipse'},
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
        content: {
          text: "",
          url: "",
          color: ""
        }
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
		  switch(item){
			  case 'square':
				  this.addNode(
					 { "content": { "text": "New Square" }, "width": 60, "height": 60, "shape": "rectangle", "point": { "x": 9.999999999999993, "y": 34.31059443007615 } } 
					) 
					  break;
					  case 'rectangle':
					  this.addNode(
					 {  "content": { "text": "New Rectangle" }, "width": 100, "height": 60, "shape": "rectangle", "point": { "x": 9.999999999999993, "y": 34.31059443007615 } } 
					) 
				  break;
				  case 'circle':
					  this.addNode(
					 { "content": { "text": "New Circle" }, "width": 60, "height": 60, "shape": "ellipse", "point": { "x": 9.999999999999993, "y": 34.31059443007615 } } 
					) 
				  break;
				  case 'oval':
					  this.addNode(
					 {  "content": { "text": "New Elipse" }, "width": 100, "height": 60, "shape": "ellipse", "point": { "x": 9.999999999999993, "y": 34.31059443007615 } } 
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
        content: {
          text: item.content.text,
          url: item.content.url,
          color: item.content.color
        },
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
      this.tmpNode.id = item.id;
      this.tmpNode.content.text = item.content.text;
      this.tmpNode.content.url = item.content.url;
      this.tmpNode.content.color = item.content.color;
      this.tmpNode.shape = item.shape;
      this.tmpNode.stroke = item.stroke;
      this.tmpNode.strokeWeight = item.strokeWeight;
      this.tmpNode.width = item.width;
      this.tmpNode.height = item.height;
      this.isModalActive = false;
      this.isEditModalActive = true;
    },
    editNode(item) {
      let tmp = this.graphData.nodes.find(x => x.id === item.id);
      tmp.content.text = item.content.text;
      tmp.content.url = item.content.url;
      tmp.content.color = item.content.color;
      tmp.shape = item.shape;
      tmp.stroke = item.stroke;
      tmp.strokeWeight = item.strokeWeight;
      tmp.width = parseInt(item.width);
      tmp.height = parseInt(item.height);
      this.isEditModalActive = false;
    },
    openLinkEdit(item) {
      this.tmpLink.id = item.id;
      this.tmpLink.content = Object.assign({}, item.content);
      this.isEditLinkModalActive = true;
    },
    editLink(item) {
      let tmp = this.graphData.links.find(x => x.id === item.id);
      tmp.color = item.content.color;
      tmp.shape = item.content.shape;
      tmp.pattern = item.content.pattern;
      tmp.arrow = item.content.arrow;
      this.isEditLinkModalActive = false;
    },
    endEdit() {
      this.editable = false;
    },
    nodeClicked(id) {
      this.$emit("nodeClicked", id);
    },
    linkClicked(id) {
      this.$emit("linkClicked", id);
    },
    nodeChanged(obj) {
      this.graphData.nodes = obj.nodes;
    },
    linkChanged(obj) {
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
