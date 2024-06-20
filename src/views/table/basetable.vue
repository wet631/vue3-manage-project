<template>
	<div>
		<TableSearch :query="query" :options="searchOpt" :search="handleSearch" />
		<div class="container">
			<TableCustom :columns="columns" :tableData="tableData" :total="page.total" :viewFunc="handleView"
				:delFunc="handleDelete" :editFunc="handleEdit" :refresh="getData" :currentPage="page.index"
				:changePage="changePage">
				<template #toolbarBtn>
					<el-button type="warning" :icon="CirclePlusFilled" @click="visible = true">新增</el-button>
				</template>
				<template #money="{ rows }">
					￥{{ rows.money }}
				</template>
				<template #thumb="{ rows }">
					<el-image class="table-td-thumb" :src="rows.thumb" :z-index="10" :preview-src-list="[rows.thumb]"
						preview-teleported>
					</el-image>
				</template>
				<template #state="{ rows }">
					<el-tag :type="rows.state ? 'success' : 'danger'">
						{{ rows.state ? '正常' : '异常' }}
					</el-tag>
				</template>
			</TableCustom>

		</div>
		<el-dialog :title="isEdit ? '编辑' : '新增'" v-model="visible" width="700px" destroy-on-close
			:close-on-click-modal="false" @close="closeDialog">
			<TableEdit :form-data="rowData" :options="options" :edit="isEdit" :update="updateData">
				<template #thumb="{ rows }">
					<img class="table-td-thumb" :src="rows.thumb"></img>
				</template>
			</TableEdit>
		</el-dialog>
		<el-dialog title="查看详情" v-model="visible1" width="700px" destroy-on-close>
			<TableDetail :data="viewData">
				<template #thumb="{ rows }">
					<el-image :src="rows.thumb"></el-image>
				</template>
			</TableDetail>
		</el-dialog>
	</div>
</template>

<script setup lang="ts" name="basetable">
import { ref, reactive } from 'vue';
import { ElMessage, } from 'element-plus';
import { CirclePlusFilled } from '@element-plus/icons-vue';
import { fetchData } from '@/api/index';
import TableCustom from '@/components/table-custom.vue';
import TableDetail from '@/components/table-detail.vue';
import TableSearch from '@/components/table-search.vue';
import { TableItem } from '@/types/table';
import { FormOption, FormOptionList } from '@/types/form-option';

// 查询相关参数
const query = reactive({
	name: '',
	dd:''
});
const selectLists = ref([
	{label:'是',value:'1'},
	{label:'否',value:'2'}
])
const cascaderOptions = ref([
  {
    value: 'guide',
    label: 'Guide',
    children: [
      {
        value: 'disciplines',
        label: 'Disciplines',
        children: [
          {
            value: 'consistency',
            label: 'Consistency',
          },
          {
            value: 'feedback',
            label: 'Feedback',
          },
          {
            value: 'efficiency',
            label: 'Efficiency',
          },
          {
            value: 'controllability',
            label: 'Controllability',
          },
        ],
      },
      {
        value: 'navigation',
        label: 'Navigation',
        children: [
          {
            value: 'side nav',
            label: 'Side Navigation',
          },
          {
            value: 'top nav',
            label: 'Top Navigation',
          },
        ],
      },
    ],
  },
  {
    value: 'component',
    label: 'Component',
    children: [
      {
        value: 'basic',
        label: 'Basic',
        children: [
          {
            value: 'layout',
            label: 'Layout',
          },
          {
            value: 'color',
            label: 'Color',
          },
          {
            value: 'typography',
            label: 'Typography',
          },
          {
            value: 'icon',
            label: 'Icon',
          },
          {
            value: 'button',
            label: 'Button',
          },
        ],
      },
      {
        value: 'form',
        label: 'Form',
        children: [
          {
            value: 'radio',
            label: 'Radio',
          },
          {
            value: 'checkbox',
            label: 'Checkbox',
          },
          {
            value: 'input',
            label: 'Input',
          },
          {
            value: 'input-number',
            label: 'InputNumber',
          },
          {
            value: 'select',
            label: 'Select',
          },
          {
            value: 'cascader',
            label: 'Cascader',
          },
          {
            value: 'switch',
            label: 'Switch',
          },
          {
            value: 'slider',
            label: 'Slider',
          },
          {
            value: 'time-picker',
            label: 'TimePicker',
          },
          {
            value: 'date-picker',
            label: 'DatePicker',
          },
          {
            value: 'datetime-picker',
            label: 'DateTimePicker',
          },
          {
            value: 'upload',
            label: 'Upload',
          },
          {
            value: 'rate',
            label: 'Rate',
          },
          {
            value: 'form',
            label: 'Form',
          },
        ],
      },
      {
        value: 'data',
        label: 'Data',
        children: [
          {
            value: 'table',
            label: 'Table',
          },
          {
            value: 'tag',
            label: 'Tag',
          },
          {
            value: 'progress',
            label: 'Progress',
          },
          {
            value: 'tree',
            label: 'Tree',
          },
          {
            value: 'pagination',
            label: 'Pagination',
          },
          {
            value: 'badge',
            label: 'Badge',
          },
        ],
      },
      {
        value: 'notice',
        label: 'Notice',
        children: [
          {
            value: 'alert',
            label: 'Alert',
          },
          {
            value: 'loading',
            label: 'Loading',
          },
          {
            value: 'message',
            label: 'Message',
          },
          {
            value: 'message-box',
            label: 'MessageBox',
          },
          {
            value: 'notification',
            label: 'Notification',
          },
        ],
      },
      {
        value: 'navigation',
        label: 'Navigation',
        children: [
          {
            value: 'menu',
            label: 'Menu',
          },
          {
            value: 'tabs',
            label: 'Tabs',
          },
          {
            value: 'breadcrumb',
            label: 'Breadcrumb',
          },
          {
            value: 'dropdown',
            label: 'Dropdown',
          },
          {
            value: 'steps',
            label: 'Steps',
          },
        ],
      },
      {
        value: 'others',
        label: 'Others',
        children: [
          {
            value: 'dialog',
            label: 'Dialog',
          },
          {
            value: 'tooltip',
            label: 'Tooltip',
          },
          {
            value: 'popover',
            label: 'Popover',
          },
          {
            value: 'card',
            label: 'Card',
          },
          {
            value: 'carousel',
            label: 'Carousel',
          },
          {
            value: 'collapse',
            label: 'Collapse',
          },
        ],
      },
    ],
  },
  {
    value: 'resource',
    label: 'Resource',
    children: [
      {
        value: 'axure',
        label: 'Axure Components',
      },
      {
        value: 'sketch',
        label: 'Sketch Templates',
      },
      {
        value: 'docs',
        label: 'Design Documentation',
      },
    ],
  },
])
const cascaderProps = ref({
  checkStrictly: true,
})
const searchOpt = ref<FormOptionList[]>([
	{ type: 'input', label: '用户名：', prop: 'name' },
	{ type: 'select', label: '大队', prop: 'dd',opts:selectLists.value,width:'200px'},
	{ type: 'date', label: '登录时间', prop: 'dlsj',format:'YYYY-MM-DD',typeTime:'datetimerange',placeholder:'请选择上报时间'},
	{ type: 'date', label: '上报时间', prop: 'sbsj',format:'YYYY-MM-DD',placeholder:'请选择上报时间'},
	{ type: 'cascader', label: '级联', prop: 'jl',placeholder:'请选择',cascaderOptions:cascaderOptions.value,cascaderProps:cascaderProps.value},
])

const handleSearch = () => {
	changePage(1);
};

// 表格相关
let columns = ref([
	{ type: 'selection' },
	{ type: 'index', label: '序号', width: 55, align: 'center' },
	{ prop: 'name', label: '用户名' },
	{ prop: 'money', label: '账户余额' },
	{ prop: 'thumb', label: '头像' },
	{ prop: 'state', label: '账户状态' },
	{ prop: 'operator', label: '操作', width: 250 },
])

const page = reactive({
	index: 1,
	size: 10,
	total: 200,
})
const tableData = ref<TableItem[]>([]);
const getData = async () => {
	const res = await fetchData()
	tableData.value = res.data.list;
};
getData();

const changePage = (val: number) => {
	page.index = val;
	getData();
};

// 新增/编辑弹窗相关
let options = ref<FormOption>({
	labelWidth: '100px',
	span: 24,
	list: [
		{ type: 'input', label: '用户名', prop: 'name', required: true },
		{ type: 'number', label: '账户余额', prop: 'money', required: true },
		{ type: 'switch', activeText: '正常', inactiveText: '异常', label: '账户状态', prop: 'state', required: true },
		{ type: 'upload', label: '头像', prop: 'thumb', required: true },
	]
})
const visible = ref(false);
const isEdit = ref(false);
const rowData = ref({});
const handleEdit = (row: TableItem) => {
	rowData.value = { ...row };
	isEdit.value = true;
	visible.value = true;
};
const updateData = () => {
	closeDialog();
	getData();
};

const closeDialog = () => {
	visible.value = false;
	isEdit.value = false;
};

// 查看详情弹窗相关
const visible1 = ref(false);
const viewData = ref({
	row: {},
	list: []
});
const handleView = (row: TableItem) => {
	viewData.value.row = { ...row }
	viewData.value.list = [
		{
			prop: 'id',
			label: '用户ID',
		},
		{
			prop: 'name',
			label: '用户名',
		},
		{
			prop: 'money',
			label: '账户余额',
		},
		{
			prop: 'state',
			label: '账户状态',
		},
		{
			prop: 'thumb',
			label: '头像',
		},
	]
	visible1.value = true;
};

// 删除相关
const handleDelete = (row: TableItem) => {
	ElMessage.success('删除成功');
}
</script>

<style scoped>
.table-td-thumb {
	display: block;
	margin: auto;
	width: 40px;
	height: 40px;
}
</style>
