<template>
  <v-dialog v-model="isActive" @clickModal="cancel">
    <transition name="item">
      <div class="block">
        <div class="inner-block"><slot></slot></div>
        <v-btn @click="ok">OK</v-btn>
        <v-btn class="danger" @click="cancel">Cancel</v-btn>
      </div>
    </transition>
  </v-dialog>
</template>
<script>
export default {
  props: {
    isActive: Boolean
  },
  methods: {
    ok() {
      this.$emit("ok", true);
    },
    cancel() {
      this.$emit("cancel");
    }
  }
};
</script>
<style lang="scss" scoped>
.inner-block {
  margin-bottom: 20px;
}
.block {
  padding: 40px;
}
input {
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
