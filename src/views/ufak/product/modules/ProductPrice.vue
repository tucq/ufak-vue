<template>
  <div>
    <a-button class="editable-add-btn" @click="handleAdd">Add</a-button>
    <a-table bordered :dataSource="specsDataSource" :columns="specsColumns">
      <template
        v-for="col in ['specsName', 'virtualPrice', 'discount','price','stock','stats']"
        :slot="col" slot-scope="text, record, index"
        :indentSize="30"
      >
        <div :key="col">
          <a-input
            v-if="record.editable"
            style="margin: -5px 0"
            :value="text"
            @change="e => handleChange(e.target.value, record.key, col)"
          />
          <template v-else
          >{{text}}</template
          >
        </div>
      </template>

      <!--      <template slot="specsName" slot-scope="text, record">-->
      <!--        <editable-cell :text="text" @change="onCellChange(record.key, 'specsName', $event)" />-->
      <!--      </template>-->

      <template slot="operation" slot-scope="text, record, index">
        <div class="editable-row-operations">
          <span v-if="record.editable">
            <a @click="() => save(record.key)">Save</a>
            <a-popconfirm title="Sure to cancel?" @confirm="() => cancel(record.key)">
              <a>Cancel</a>
            </a-popconfirm>
          </span>
          <span v-else>
            <a @click="() => edit(record.key)">Edit</a>
          </span>

          <a-divider type="vertical"/>

          <a-popconfirm
            v-if="specsDataSource.length"
            title="Sure to delete?"
            @confirm="() => onDelete(record.key)"
          >
            <a href="javascript:;">Delete</a>
          </a-popconfirm>

        </div>
      </template>

      <!--      <template slot="operation" slot-scope="text, record">-->
      <!--        <a-popconfirm-->
      <!--          v-if="specsDataSource.length"-->
      <!--          title="Sure to delete?"-->
      <!--          @confirm="() => onDelete(record.key)"-->
      <!--        >-->
      <!--          <a href="javascript:;">Delete</a>-->
      <!--        </a-popconfirm>-->
      <!--      </template>-->
    </a-table>
  </div>
</template>

<script>
    import EditableCell from '@/views/ufak/common/EditableCell';

    const data = [
        {
            key: '1',
            specsName: '白色',
            specsImage: '32',
            virtualPrice: '200',
            discount: '1',
            price: '200',
            stock: '100',
            stats: '启用',
        },
        {
            key: '2',
            specsName: '红色',
            specsImage: '20',
            virtualPrice: '300',
            discount: '1',
            price: '200',
            stock: '100',
            stats: '启用',
        },
    ];

    export default {
        name: "ProductSpecs",
        components: {
            EditableCell,
        },
        data() {
            this.cacheData = data.map(item => ({ ...item }));
            return {
                specsDataSource: data,
                count: 2,
                specsColumns: [
                    {
                        title: '规格名称',
                        dataIndex: 'specsName',
                        width: '20%',
                        scopedSlots: { customRender: 'specsName' },
                    },
                    {
                        title: '规格图片',
                        width: '15%',
                        dataIndex: 'specsImage',
                    },
                    {
                        title: '虚拟价',
                        width: '10%',
                        dataIndex: 'virtualPrice',
                        scopedSlots: { customRender: 'virtualPrice' },

                    },
                    {
                        title: '折扣',
                        width: '10%',
                        dataIndex: 'discount',
                        scopedSlots: { customRender: 'discount' },
                    },
                    {
                        title: '售卖价',
                        width: '10%',
                        dataIndex: 'price',
                        scopedSlots: { customRender: 'price' },
                    },
                    {
                        title: '库存',
                        width: '10%',
                        dataIndex: 'stock',
                        scopedSlots: { customRender: 'stock' },
                    },
                    {
                        title: '启/停用',
                        width: '10%',
                        dataIndex: 'stats',
                        scopedSlots: { customRender: 'stats' },
                    },
                    {
                        title: '操作',
                        width: '15%',
                        dataIndex: 'operation',
                        scopedSlots: { customRender: 'operation' },
                    },
                ],
            };
        },
        methods: {
            handleChange(value, key, column) {
                const newData = [...this.specsDataSource];
                const target = newData.filter(item => key === item.key)[0];
                if (target) {
                    target[column] = value;
                    this.specsDataSource = newData;
                }
            },
            edit(key) {
                const newData = [...this.specsDataSource];
                const target = newData.filter(item => key === item.key)[0];
                if (target) {
                    target.editable = true;
                    this.specsDataSource = newData;
                }
            },
            save(key) {
                const newData = [...this.specsDataSource];
                const newCacheData = [...this.cacheData];
                const target = newData.filter(item => key === item.key)[0];
                const targetCache = newCacheData.filter(item => key === item.key)[0];
                if (target && targetCache) {
                    delete target.editable;
                    this.specsDataSource = newData;
                    Object.assign(
                        targetCache,
                        target
                    );
                    this.cacheData = newCacheData;
                }
            },
            cancel(key) {
                const newData = [...this.specsDataSource];
                const target = newData.filter(item => key === item.key)[0];
                if (target) {
                    Object.assign(target, this.cacheData.filter(item => key === item.key)[0]);
                    delete target.editable;
                    this.specsDataSource = newData;
                }
            },

            onCellChange(key, dataIndex, value) {
                const specsDataSource = [...this.specsDataSource];
                const target = specsDataSource.find(item => item.key === key);
                if (target) {
                    target[dataIndex] = value;
                    this.specsDataSource = specsDataSource;
                }
            },
            onDelete(key) {
                const specsDataSource = [...this.specsDataSource];
                this.specsDataSource = specsDataSource.filter(item => item.key !== key);
            },
            handleAdd() {
                const { count, specsDataSource } = this;
                const newData = {
                    key: count,
                    name: `Edward King ${count}`,
                    age: 32,
                    address: `London, Park Lane no. ${count}`,
                };
                this.specsDataSource = [...specsDataSource, newData];
                this.count = count + 1;
            },
        },
    }
</script>

<style scoped>
  .editable-cell {
    position: relative;
  }

  .editable-cell-input-wrapper,
  .editable-cell-text-wrapper {
    padding-right: 24px;
  }

  .editable-cell-text-wrapper {
    padding: 5px 24px 5px 5px;
  }

  .editable-cell-icon,
  .editable-cell-icon-check {
    position: absolute;
    right: 0;
    width: 20px;
    cursor: pointer;
  }

  .editable-cell-icon {
    line-height: 18px;
    display: none;
  }

  .editable-cell-icon-check {
    line-height: 28px;
  }

  .editable-cell:hover .editable-cell-icon {
    display: inline-block;
  }

  .editable-cell-icon:hover,
  .editable-cell-icon-check:hover {
    color: #108ee9;
  }

  .editable-add-btn {
    margin-bottom: 8px;
  }


  /********************/
  .editable-row-operations a {
    margin-right: 8px;
  }
</style>