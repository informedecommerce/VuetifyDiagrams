<template>
  
    <transition name="item">
      <div class="form" v-if="isActive">
        <v-text-field v-model="newNode.content.text" placeholder="name" /><br />
        <v-text-field v-model="newNode.content.url" placeholder="url" /><br />
        <v-text-field v-model="newNode.content.color" placeholder="color" /><br />
        <v-text-field
          type="number"
          v-model="newNode.width"
          placeholder="width"
        /><br />
        <v-text-field
          type="number"
          v-model="newNode.height"
          placeholder="height"
        /><br />
        <v-text-field
          type="text"
          v-model="newNode.stroke"
          placeholder="stroke"
        /><br />
        <v-text-field
          type="number"
          v-model="newNode.strokeWeight"
          placeholder="stroke weight"
        /><br />
        <v-select v-model="newNode.shape" placeholder="Select shape">
          <option value="rectangle" selected>Rectangle</option>
          <option value="ellipse">Ellipse</option> </v-select
        ><br />
        <v-btn @click="ok">OK</v-btn>
        <v-btn class="danger" @click="cancel">Cancel</v-btn>
      </div>
    </transition>

</template>
<script>
export default {
  props: {
    isActive: Boolean,
    node: {
      type: Object,
      default() {
        return {
          id: "",
          shape: "rectangle",
          width: 150,
          height: 60,
          stroke: "",
          strokeWeight: 0,
          content: {
            text: "none",
            url: "",
            color: "#ecf0f1"
          }
        };
      }
    }
  },
  watch: {
    node() {
      this.newWidth = parseInt(this.node.width);
      this.newHeight = parseInt(this.node.Height);
    }
  },
  data() {
    return {
      newNode: this.node
    };
  },
  methods: {
    ok() {
      this.$emit("ok", this.newNode);
    },
    cancel() {
      this.$emit("cancel");
    }
  }
};
</script>
<style lang="scss" scoped>
input {
  width: 95%;
  margin-bottom: 5px;
}
select {
  margin-bottom: 5px;
}
.item-enter-active {
  transition: all 0.8s ease;
}
.item-leave-active {
  transition: all 0.3s ease;
}
.item-enter,
.item-leave-to {
  opacity: 0;
}
</style>
