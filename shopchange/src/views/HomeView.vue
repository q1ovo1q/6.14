<template>
  <div id="app">
    <el-form @submit.prevent="sendData" ref="form" label-width="100px" class="form-container"
      style="max-width:1200px;margin:0 auto;">
      <el-form-item label="名称">
        <el-input v-model="goods_name"></el-input>
      </el-form-item>
      <el-form-item label="库存">
        <el-input v-model="stock"></el-input>
      </el-form-item>
      <el-form-item label="价格">
        <el-input v-model="price"></el-input>
      </el-form-item>
      <el-form-item label="售价">
        <el-input v-model="sold"></el-input>
      </el-form-item>
      <el-form-item label="图片">
        <el-input v-model="img"></el-input>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="sendData">提交</el-button>
      </el-form-item>
    </el-form>

    <div class="search-container" style="margin: 20px auto;width: 1200px;">
      <el-input v-model="searname" placeholder="根据id获取商品详情"></el-input>
      <el-button type="primary" @click="seek">查询</el-button>
    </div>

    <el-table :data="list" tooltip-effect="dark" @selection-change="handleSelectionChange" ref="multipleTable"
      class="table-container" style="margin: 20px auto;width: 1300px;">
      <el-table-column type="selection" width="66"></el-table-column>
      <el-table-column label="ID" width="100" prop="id"></el-table-column>
      <el-table-column label="名称" width="100" prop="goods_name"></el-table-column>
      <el-table-column label="库存" width="60" prop="stock"></el-table-column>
      <el-table-column label="价格" width="100" prop="price"></el-table-column>
      <el-table-column label="状态" width="50" prop="status"></el-table-column>
      <el-table-column label="售价" width="100" prop="sold"></el-table-column>
      <el-table-column label="图片" width="100">
        <template slot-scope="scope">
          <img v-if="scope.row && scope.row.img" :src="scope.row.img" alt="" style="width: 100px; height: auto;">
        </template>
      </el-table-column>
      <el-table-column label="创建时间" width="160" prop="created_at"></el-table-column>
      <el-table-column label="更新时间" width="160" prop="updated_at"></el-table-column>
      <el-table-column label="操作" width="260">
        <template slot-scope="scope">
          <!-- 编辑按钮 -->
          <el-button type="primary" icon="el-icon-edit" @click="editItem(scope.row)" size="small">编辑</el-button>
          <!-- 删除按钮 -->
          <el-button type="danger" icon="el-icon-delete" @click="deleteItem(scope.row.id)" size="small">删除</el-button>
          <!-- 查看按钮 -->
          <el-button type="info" icon="el-icon-view" @click="viewItem(scope.row)" size="small">查看</el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 商品详情模态框 -->
    <el-dialog :visible.sync="dialogVisible" title="商品详情" width="30%">
      <div>
        <p>ID: {{ selectedItem.id }}</p>
        <p>名称: {{ selectedItem.goods_name }}</p>
        <p>库存: {{ selectedItem.stock }}</p>
        <p>价格: {{ selectedItem.price }}</p>
        <!-- 其他字段 -->
      </div>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">关闭</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import axios from 'axios';
import dayjs from 'dayjs';

export default {
  name: 'HomeView',
  data() {
    return {
      dialogVisible: false, // 控制模态框显示
      selectedItem: {}, // 选中的商品信息
      id: '',
      goods_name: '',
      stock: '',
      price: '',
      status: '',
      sold: '',
      img: '',
      created_at: '',
      updated_at: '',
      list: [],
      searname: ''
    }
  },
  methods: {
    listData() {
      axios.get('https://liu.zzgoodqc.cn/goodsx/goodslist')
        .then(response => {
          console.log("初始化数据：", response.data.data);
          this.list = response.data.data;
        })
        .catch(err => {
          console.log("获取数据报错！", err);
        });
    },
    // 查看商品详情
    viewItem(item) {
      this.selectedItem = item; // 设置选中的商品信息
      this.dialogVisible = true; // 打开模态框
    },
    handleSelectionChange(val) {
      this.multipleSelection = val;
    },
    sendData() {
      if (this.id) {
        this.updateItem();
      } else {
        if (!this.img) {
          this.img = '';
        }
        this.addItem();
      }
    },
    // 添加商品
    addItem() {
      let obj = {
        goods_name: this.goods_name,
        price: Number(this.price),  // 确保价格是数字
        stock: Number(this.stock),  // 确保库存是数字
        status: this.status,
        sold: this.sold,
        img: this.img || '',
      };
      console.log("即将发送的数据：", obj);

      axios.post("https://liu.zzgoodqc.cn/goodsx/add", obj)
        .then((response) => {
          console.log("服务器响应数据：", response.data);

          // 检查响应是否成功
          if (response.data && response.data.code === 0) { // 根据实际情况修改判断条件
            // 添加成功
            this.listData(); // 重新获取列表数据
            this.clearForm();
            this.$message.success("添加成功！");
          } else {
            console.error("添加失败:", response.data.msg);
            this.$message.error("添加失败: " + response.data.msg);
          }
        })
        .catch((err) => {
          console.error("添加报错: ", err);
          this.$message.error("添加报错: " + err.message);
        });
    },
    updateItem() {
      const now = dayjs().format('YYYY-MM-DD HH:mm:ss');
      const updatedItem = {
        id: this.id,
        goods_name: this.goods_name,
        price: this.price,
        stock: this.stock,
        status: this.status,
        sold: this.sold,
        img: this.img,
        created_at: this.created_at,
        updated_at: now,
      };
      axios.post(`https://liu.zzgoodqc.cn/goodsx/update?id=${this.id}`, updatedItem)
        .then((res) => {
          const index = this.list.findIndex(item => item.id === this.id);
          if (index !== -1) {
            this.$set(this.list, index, res.data.data);
          }
          this.clearForm();
        })
        .catch((error) => {
          console.error("更新报错！", error);
        });
    },
    seek() {
      axios.post(`https://liu.zzgoodqc.cn/goodsx/search/?goods_name=${this.searname}`)
        .then((res) => {
          this.list = [res.data.data];
        })
        .catch((error) => {
          console.error("查询报错！", error);
        });
    },
    deleteItem(id) {
      axios.get(`https://liu.zzgoodqc.cn/goodsx/delete?id=${id}`)
        .then(() => {
          this.list = this.list.filter(item => item.id !== id);
        })
        .catch((error) => {
          console.error("删除报错！", error);
        });
    },
    editItem(item) {
      this.id = item.id;
      this.goods_name = item.goods_name;
      this.stock = item.stock;
      this.price = item.price;
      this.status = item.status;
      this.sold = item.sold;
      this.img = item.img;
      this.created_at = item.created_at;
      this.updated_at = item.updated_at;
    },
    clearForm() {
      this.id = '';
      this.goods_name = '';
      this.stock = '';
      this.price = '';
      this.status = '';
      this.sold = '';
      this.img = '';
      this.created_at = '';
      this.updated_at = '';
      if (this.$refs.form) {
        this.$refs.form.resetFields();  // 重置表单字段
      }
    }
  },
  mounted() {
    this.listData();
  }
}
</script>

<style scoped>
#app {
  margin: 0 auto;
  width: 100%;
  max-width: 1800px;
}

.form-container {
  max-width: 1200px;
  margin: 0 auto;
  background: #f9f9f9;
  padding: 20px;
  margin-bottom: 20px;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.table-container {
  max-width: 1300px;
  margin: 20px auto;
}

.el-button {
  margin-right: 10px;
  margin-bottom: 10px;
}

.search-container {
  display: flex;
  align-items: center;
}

.search-container .el-input {
  flex: 1;
}

.table-container {
  background: #fff;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}
</style>
