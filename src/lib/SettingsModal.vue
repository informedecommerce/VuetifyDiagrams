<template>
  <div>
    <h2>Settings</h2>
    <h3>Field Size</h3>
    <label>Width</label>
    <v-text-field type="number" v-model="newSettings.width" />
    <label>Height</label>
    <v-text-field type="number" v-model="newSettings.height" /><br />
    <h3>Scale</h3>
    <v-select lebel="Scale" v-model="newSettings.scale" :items="size_options" item-text="text" item-value="value">
    </v-select>
    <h3>Fluid</h3>
	  <v-switch
      v-model="newSettings.isFluid"
      label="Toggle fluid"
    ></v-switch>
    <h3>Background</h3>
	  <v-switch
      v-model="newSettings.showGrid"
      label="Show grid"
    ></v-switch>
    <v-btn @click="ok">OK</v-btn>
    <v-btn class="danger" @click="cancel">Cancel</v-btn>
</div>
</template>
<script>
export default {
  props: {
    isActive: {
      type: Boolean,
      default: false
    },
    settings: {
      type: Object,
      default() {
        return {
          width: 1500,
          height: 1000,
          scale: "1",
          isFluid: false,
          showGrid: false
        };
      }
    }
  },
  watch: {
    isActive(val) {
      if (val) {
        this.newSettings = Object.assign({}, this.settings);
      }
    }
  },
  data() {
    return {
      newSettings: {
        width: 0,
        height: 0,
        showGrid: false
      },
		size_options: [
			{
				text: 'Small',
				val: 0.5
			},
			{
				text: 'Medium',
				val: 1
			},
			{
				text: 'Large',
				val: 1.5
			}
		]
    };
  },
  methods: {
    changeGrid() {
      this.$emit("changeGrid");
    },
    ok() {
      this.$emit("ok", this.newSettings);
    },
    cancel() {
      this.$emit("cancel");
    }
  }
};
</script>
