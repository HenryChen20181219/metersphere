<template>
  <ms-test-plan-common-component>
    <template v-slot:aside>
      <node-tree
        class="node-tree"
        v-loading="result.loading"
        @nodeSelectEvent="nodeChange"
        :tree-nodes="treeNodes"
        ref="nodeTree"/>
    </template>
    <template v-slot:main>
      <test-plan-load-case-list
        class="table-list"
        @refresh="refresh"
        :review-id="reviewId"
        :clickType="clickType"
        :select-project-id="selectProjectId"
        :select-parent-nodes="selectParentNodes"
        :version-enable="versionEnable"
        @relevanceCase="openTestCaseRelevanceDialog"
        ref="testPlanLoadCaseList"/>
    </template>
    <test-case-load-relevance
      @refresh="refresh"
      :review-id="reviewId"
      :version-enable="versionEnable"
      ref="testCaseLoadRelevance"/>
  </ms-test-plan-common-component>

</template>

<script>
import MsTestPlanCommonComponent from "@/business/components/track/plan/view/comonents/base/TestPlanCommonComponent";
import NodeTree from "@/business/components/track/common/NodeTree";
import TestPlanLoadCaseList from "@/business/components/track/plan/view/comonents/load/TestPlanLoadCaseList";
import TestCaseLoadRelevance from "@/business/components/track/plan/view/comonents/load/TestCaseLoadRelevance";
import {hasLicense} from "@/common/js/utils";

export default {
  name: "TestReviewLoad",
  components: {
    MsTestPlanCommonComponent,
    NodeTree,
    TestPlanLoadCaseList,
    TestCaseLoadRelevance,
  },
  data() {
    return {
      result: {},
      selectNodeIds: [],
      selectParentNodes: [],
      selectProjectId: "",
      projectId: null,
      treeNodes: [],
      versionEnable: false,
    }
  },
  props: [
    'reviewId',
    'redirectCharType',
    'clickType'
  ],
  watch: {
    planId() {
      this.initData();
    }
  },
  mounted() {
    this.initData();
    this.checkVersionEnable();
  },
  methods: {
    refresh() {
      this.selectProjectId = '';
      this.selectParentNodes = [];
      this.$refs.testPlanLoadCaseList.initTable();
      this.getNodeTreeByPlanId();
    },
    initData() {
      this.getNodeTreeByPlanId();
    },
    openTestCaseRelevanceDialog() {
      this.$refs.testCaseLoadRelevance.open();
    },
    nodeChange(node, nodeIds, pNodes) {
      this.selectProjectId = node.key;
      // 切换node后，重置分页数
      this.$refs.testPlanLoadCaseList.currentPage = 1;
      this.$refs.testPlanLoadCaseList.pageSize = 10;
    },
    getNodeTreeByPlanId() {
      if (this.planId) {
        this.result = this.$get("/case/node/list/plan/" + this.planId, response => {
          this.treeNodes = response.data;
          // 性能测试与模块无关，过滤项目下模块
          this.treeNodes.map(node => node.children = null);
        });
      }
    },
    checkVersionEnable() {
      if (!this.projectId) {
        return;
      }
      if (hasLicense()) {
        this.$get('/project/version/enable/' + this.projectId, response => {
          this.versionEnable = response.data;
        });
      }
    },
  }
}
</script>

<style scoped>

</style>
