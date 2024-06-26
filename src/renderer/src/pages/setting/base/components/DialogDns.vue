<template>
  <t-dialog v-model:visible="formVisible" header="DNS-over-HTTP" placement="center" :footer="false">
    <template #body>
      <div class="doh-dialog-container dialog-container-padding">
        <div class="header">
          <p class="tip">确定如何通过安全连接来连接到网站</p>
        </div>

        <!-- 表单内容 -->
        <t-form ref="form" :data="formData" @submit="onSubmit">
          <t-textarea v-model="formData.data" class="dns-input" placeholder="请输入dns" autofocus
            :autosize="{ minRows: 2, maxRows: 4 }" @change="changeDnstextarea" />
          <t-radio-group v-model="dnsSelect" variant="default-filled" size="small" @change="changeDnsSelect">
            <t-radio-button v-for="item in DNS_LIST" :key="item.name" :value="item.dns">{{ item.name }}</t-radio-button>
          </t-radio-group>
          <p class="tip bottom-tip">推荐使用腾讯国密级，为空使用普通dns查询</p>
          <div class="optios">
            <t-form-item style="float: right">
              <t-button variant="outline" @click="onClickCloseBtn">取消</t-button>
              <t-button theme="primary" type="submit">确定</t-button>
            </t-form-item>
          </div>
        </t-form>
      </div>
    </template>
  </t-dialog>
</template>

<script setup lang="ts">
import _ from 'lodash';
import { MessagePlugin } from 'tdesign-vue-next';
import { reactive, ref, watch } from 'vue';

import DNS_CONFIG from '@/config/doh';

const props = defineProps({
  visible: {
    type: Boolean,
    default: false,
  },
  data: {
    type: Object,
    default: () => {
      return {
        data: '',
        type: '',
      };
    },
  },
});
const formVisible = ref(false);
const formData = ref(props.data);

const DNS_LIST = reactive([...DNS_CONFIG.doh]);

const dnsSelect = ref('');

const emit = defineEmits(['update:visible', 'receiveDnsData']);

watch(
  () => formVisible.value,
  (val) => {
    emit('update:visible', val);
  },
);
watch(
  () => props.visible,
  (val) => {
    formVisible.value = val;
  },
);
watch(
  () => props.data,
  (val) => {
    formData.value = val;

    const index = _.findIndex(DNS_LIST, ['dns', val.data]);
    if (index > -1) dnsSelect.value = val.data;
  },
);

const changeDnstextarea = (item) => {
  const index = _.findIndex(DNS_LIST, ['dns', item]);
  if (index === -1) dnsSelect.value = null;
};

const changeDnsSelect = (item) => {
  formData.value.data = item;
};

const onSubmit = async () => {
  const { data, type } = formData.value;
  emit('receiveDnsData', {
    data,
    type,
  });

  window.electron.ipcRenderer.send('update-dns', !!data, data);
  MessagePlugin.info('重启软件生效');

  formVisible.value = false;
};

const onClickCloseBtn = () => {
  formVisible.value = false;
};
</script>

<style lang="less" scoped></style>
