<template>
 
    <div>
      <v-textarea
        class="input"
        v-model="tmp"
        required
        placeholder="JSON"
      ></v-textarea
      ><br />
      <v-btn @click="ok">OK</v-btn>
      <v-btn class="danger" @click="cancel">Cancel</v-btn>
    </div>
  
</template>
<script>
export default {
  props: {
    isActive: Boolean,
    text: String
  },
  watch: {
    text(val) {
      this.tmp = val;
    }
  },
  data() {
    return {
      tmp: ""
    };
  },
  methods: {
    ok() {
      this.$emit("ok", {
        text: this.tmp
      });
    },
    cancel() {
      this.$emit("cancel");
    }
  }
};
</script>
<style lang="scss" scoped>
.input {
  width: 100%;
  height: 100px;
  margin-bottom: 5px;
}
</style>
