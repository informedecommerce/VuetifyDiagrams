<template>
  <input :type="type" v-model="val" @input="onInput" />
</template>
<script>
export default {
  name: "v-textfield",
  props: {
    type: String,
    value: [String, Number]
  },
  computed: {
    val: {
      get() {
        return this.value;
      },
      set(content) {
        this.$emit("input", content);
      }
    }
  },
  methods: {
    onInput() {
      if (this.value !== this.val) this.$emit("input", this.val);
    }
  }
};
</script>
<style lang="scss" scoped>
input {
  background: transparent;
  border: 0;
  border-bottom: 1px solid #e0e0e0;
  border-radius: 0;
  padding: 10px 5px;
}
input:focus {
  border-bottom: 1px solid #00b894;
}
input::-webkit-input-placeholder {
  font-weight: lighter;
  font-size: 20;
}
input:-moz-placeholder {
  font-weight: lighter;
  font-size: 20;
}
input::-moz-placeholder {
  font-weight: lighter;
  font-size: 20;
}
input:-ms-input-placeholder {
  font-weight: lighter;
  font-size: 20;
}
</style>
