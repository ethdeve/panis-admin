<script setup lang="tsx">
import { computed, reactive, watch } from 'vue';
import { NButton, NPopconfirm } from 'naive-ui';
import { fetchDeleteDictItem, fetchGetDictItemPageList } from '@/service/api';
import { useAppStore } from '@/store/modules/app';
import { useTable, useTableOperate } from '@/hooks/common/table';
import { useAuth } from '@/hooks/business/auth';
import { useDict } from '@/hooks/business/dict';
import { transDeleteParams } from '@/utils/common';
import { $t } from '@/locales';
import DictItemSearch from './dict-item-search.vue';
import DictItemOperateDrawer from './dict-item-operate-drawer.vue';

defineOptions({
  name: 'DictItemPageListTable'
});

interface Props {
  dict: Api.SystemManage.DictTree;
}

const props = defineProps<Props>();
const dictId = computed(() => props.dict?.id);

const appStore = useAppStore();

const { hasAuth } = useAuth();

const { dictTag } = useDict();

/** api params */
const apiParams = reactive({
  page: 1,
  pageSize: 20,
  dictId: dictId.value,
  value: null,
  description: null,
  zhCN: null,
  enUS: null
});

const { columns, columnChecks, data, loading, getData, getDataByPage, mobilePagination, searchParams, updateSearchParams, resetSearchParams } =
  useTable({
    apiFn: fetchGetDictItemPageList,
    apiParams,
    columns: () => [
      {
        type: 'selection',
        align: 'center',
        width: 48
      },
      {
        key: 'index',
        title: $t('common.index'),
        width: 64,
        align: 'center'
      },
      {
        key: 'value',
        title: $t('page.manage.dictItem.value'),
        minWidth: 64,
        align: 'center'
      },
      {
        key: 'zhCN',
        title: $t('page.manage.dictItem.zhCN'),
        minWidth: 64,
        align: 'center'
      },
      {
        key: 'enUS',
        title: $t('page.manage.dictItem.enUS'),
        minWidth: 64,
        align: 'center'
      },
      {
        key: 'sort',
        title: $t('page.manage.dictItem.sort'),
        minWidth: 64,
        align: 'center'
      },
      {
        key: 'status',
        title: $t('page.manage.dictItem.status'),
        align: 'center',
        width: 80,
        render: row => dictTag('status', row.status)
      },
      {
        key: 'description',
        title: $t('page.manage.dictItem.description'),
        minWidth: 64,
        align: 'center',
        ellipsis: {
          tooltip: true
        }
      },

      {
        key: 'operate',
        title: $t('common.operate'),
        align: 'center',
        width: 140,
        fixed: 'right',
        render: row => (
          <div class="flex-center gap-8px">
            {hasAuth('sys:dict:item:update') && (
              <NButton type="primary" quaternary size="small" onClick={() => edit(row.id)}>
                {$t('common.edit')}
              </NButton>
            )}
            {hasAuth('sys:dict:item:delete') && (
              <NPopconfirm onPositiveClick={() => handleDelete(row.id)}>
                {{
                  default: () => $t('common.confirmDelete'),
                  trigger: () => (
                    <NButton type="error" quaternary size="small">
                      {$t('common.delete')}
                    </NButton>
                  )
                }}
              </NPopconfirm>
            )}
          </div>
        )
      }
    ]
  });

const { drawerVisible, operateType, handleAdd, handleEdit, editingId, checkedRowKeys, onDeleted, onBatchDeleted } = useTableOperate(data, getData);

/** edit */
function edit(id: string) {
  handleEdit(id);
}

/** delete */
async function handleDelete(id: string) {
  // request
  const { error, data: result } = await fetchDeleteDictItem(transDeleteParams([id]));
  if (!error && result) {
    onDeleted();
  }
}

/** batch delete */
function handleBatchDelete() {
  onBatchDeleted();
}

/** watch dictId */
watch(dictId, () => {
  updateSearchParams({ dictId: dictId.value });
  // update apiParams
  apiParams.dictId = dictId.value;
  getDataByPage();
});
</script>

<template>
  <div class="h-full flex-col-stretch gap-8px overflow-hidden lt-sm:overflow-auto">
    <DictItemSearch v-model:model="searchParams" @reset="resetSearchParams" @search="getDataByPage" />
    <NCard
      :title="dict.name + `(` + dict.code + `) ` + $t('page.manage.dictItem.title')"
      :bordered="false"
      class="sm:flex-1-hidden card-wrapper"
      content-class="flex-col"
    >
      <TableHeaderOperation
        v-model:columns="columnChecks"
        :checked-row-keys="checkedRowKeys"
        :loading="loading"
        add-auth="sys:dict:item:add"
        delete-auth="sys:dict:item:delete"
        @add="handleAdd"
        @delete="handleBatchDelete"
        @refresh="getData"
      />
      <NDataTable
        v-model:checked-row-keys="checkedRowKeys"
        remote
        striped
        size="small"
        class="sm:h-full"
        :data="data"
        :scroll-x="962"
        :columns="columns"
        :flex-height="!appStore.isMobile"
        :loading="loading"
        :single-line="false"
        :row-key="row => row.id"
        :pagination="mobilePagination"
      />
    </NCard>
    <DictItemOperateDrawer :id="editingId" v-model:visible="drawerVisible" :dict="dict" :operate-type="operateType" @submitted="getDataByPage" />
  </div>
</template>
