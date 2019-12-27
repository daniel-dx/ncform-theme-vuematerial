<template>

  <div class="__array-tabs-form-item">

    <legend v-if="schema.ui.legend && schema.ui.showLegend" @click="collapse()">
      {{_analyzeVal(schema.ui.legend)}}
      <i v-if="!mergeConfig.disableCollapse" class="fa" :class="{'fa-angle-down': !collapsed, 'fa-angle-up': collapsed}"></i>
    </legend>

    <md-tabs v-show="!collapsed" :md-active-tab="activeName">
      <template slot="md-tab" slot-scope="{ tab }">
        <span>
          {{_analyzeVal(tab.data.dataItem.__dataSchema.ui.label) + ' ' + (tab.data.idx + 1)}}
          <!-- 提示信息 -->
          <div style="display: inline-block">
            <a class="help" v-if="tab.data.dataItem.__dataSchema.ui.help.show === true" href="#"><span :class="tab.data.dataItem.__dataSchema.ui.help.iconCls">{{tab.data.dataItem.__dataSchema.ui.help.text}}</span></a>
            <md-tooltip md-direction="top">{{tab.data.dataItem.__dataSchema.ui.help.content}}</md-tooltip>
          </div>

          <span v-if="(!mergeConfig.disableDel && !isDelExceptionRow(tab.data.dataItem.__dataSchema)) || (mergeConfig.disableDel && isDelExceptionRow(tab.data.dataItem.__dataSchema))" @click.stop="handleTabsEdit(tab.data.idx + '', 'remove')" >
            <i class="fa fa-times"></i>
          </span>
        </span>
      </template>

      <md-tab v-for="(dataItem, idx) in schema.value" :key="dataItem.__dataSchema.__id" :closable="(!mergeConfig.disableDel && !isDelExceptionRow(dataItem.__dataSchema)) || (mergeConfig.disableDel && isDelExceptionRow(dataItem.__dataSchema))" :md-template-data="{ dataItem: dataItem, idx: idx, fieldSchema: fieldSchema }" :id="'' + idx">
        <!-- array item 是 正常的 object 类型 -->
          <template v-if="isNormalObjSchema(dataItem.__dataSchema)">
            <ncform-object :schema="dataItem.__dataSchema" :form-data="formData" :idx-chain="(idxChain ? idxChain + ',' : '') + idx" :config="dataItem.__dataSchema.ui.widgetConfig" :show-legend="false">

              <template v-for="(fieldSchema, fieldName) in (dataItem.__dataSchema.properties || {__notObjItem: dataItem.__dataSchema})" :slot="fieldName"><!-- 注意：__notObjItem 这个Key为与form-item约定好的值，其它名字不生效 -->
                <slot :name="fieldName" :schema="fieldSchema" :idx="idx"></slot>
              </template>

            </ncform-object>
          </template>

          <!-- array item 是 非正常的 object 类型 以及 其它类型 -->
          <template v-else>
            <slot name="__notObjItem" :schema="dataItem.__dataSchema" :idx="idx"></slot> <!-- 注意：__notObjItem 和 __dataSchema 都是约定好的值，其它名字不生效 -->
          </template>
      </md-tab>
    </md-tabs>

    <md-button v-if="!mergeConfig.disableAdd" class="md-icon-button add-btn" @click="handleTabsEdit(null, 'add')">
      <i class="fa fa-plus"></i>
    </md-button>

  </div>

</template>

<style lang="scss">
  .__array-tabs-form-item {

    // margin-top: 8px;
    position: relative;

    & > legend {
      border-left: 6px solid rgba(0, 0, 0, 0.12);
      padding: 6px;
      background-color: rgb(68, 138, 255);
      color: #fff;
      font-size: 14px;
      margin-bottom: 0px;
      border-radius: 4px 4px 0 0;
    }

    .add-btn {
      position: absolute;
      right: -6px;
      top: 35px;
    }
  }
</style>

<script>

  import ncformCommon from '@ncform/ncform-common';

  const layoutArrayMixin = ncformCommon.mixins.vue.layoutArrayMixin;

  export default {

    mixins: [layoutArrayMixin],

    i18nData: {
      en: {
        delItemTips: 'Are you sure to delete this item?',
      },
      zh_cn: {
        delItemTips: '确定要删除该项吗？',
      }
    },

    data() {
      return {
        activeName: '0',
        defaultConfig: {
          tabPosition: 'top',
        },
      }
    },

    methods: {
      handleTabsEdit(targetName, action) {
        if (action === 'add') {
          this.addItem();
          this.$data.activeName = (this.schema.value.length - 1) + '';
        }
        if (action === 'remove') {
          this.delItem(targetName, this.mergeConfig.requiredDelConfirm, this.mergeConfig.delConfirmText.item || this.$nclang('delItemTips'));
          this.$nextTick(() => {
            let tabIdx = parseInt(targetName);
            if (targetName === this.$data.activeName) { // Remote item is the active item
              if (tabIdx === 0) { // First item
                this.$data.activeName = '0'
              } else {
                this.$data.activeName = (tabIdx - 1) + '';
              }
            } else {
              let activeIdx = parseInt(this.$data.activeName);
              if (activeIdx > tabIdx) { // active item at the right side
                this.$data.activeName = (activeIdx - 1) + '';
              }
            }
          })
        }
      }
    }

  }
</script>
