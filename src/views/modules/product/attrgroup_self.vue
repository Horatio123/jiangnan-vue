<template>
  <el-row :gutter="20">
    <el-col :span="6">
      <category></category>
    </el-col>
    <el-col :span="18">
      表格
    </el-col>
  </el-row>
</template>
<script>
import Category from '../common/category'
export default {
  methods: {},
  components: {Category},
  props: {},
  data() {
    return {};
  },
  activated() {
  },
};
</script>
<style scoped>
</style>