<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <!-- 搜索与添加区 -->
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input
            placeholder="请输入内容"
            clearable
            v-model="queryInfo.query"
            @clear="getGoodsList"
          >
            <el-button slot="append" icon="el-icon-search"  @click="getGoodsList"></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="goAddpage">添加商品</el-button>
        </el-col>
      </el-row>
      <!-- 商品列表区域 -->
      <el-table border stripe :data="goodslist">
        <el-table-column type="index"></el-table-column>
        <el-table-column label="商品名称" prop="goods_name"></el-table-column>
        <el-table-column label="商品价格(元)" prop="goods_price" width="95px"></el-table-column>
        <el-table-column label="商品重量" prop="goods_weight" width="70px"></el-table-column>
        <el-table-column label="创建时间" prop="add_time" width="140px">
            <template slot-scope="scope">
                {{scope.row.add_time | dateFormat}}

            </template>
        </el-table-column>
        <el-table-column label="操作" width="140px">
          <template slot-scope="scope">
            <!-- 修改按钮 -->
            <el-button
              type="primary"
              icon="el-icon-edit"
              size="mini"
            ></el-button>
            <!-- 删除按钮 -->
            <el-button
              type="danger"
              icon="el-icon-delete"
              size="mini"
              @click="removeGoodsById(scope.row.goods_id)"
            ></el-button>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页区域 -->
       <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="queryInfo.pagenum"
      :page-sizes="[5, 10, 15, 20]"
      :page-size="queryInfo.pagesize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total" background>
    </el-pagination>
    </el-card>
  </div>
</template>

<script>
export default {
  data() {
    return {
      //查询参数对象
      queryInfo: {
        query: '',
        //当前页数
        pagenum: 1,
        //当前每页显示多少条数据
        pagesize: 10,
      },
      //商品列表
      goodslist: [],
      //总数据条数
      total: 0,
    }
  },
  created() {
    this.getGoodsList()
  },
  methods: {
      //根据分页获取对应的商品列表
    async getGoodsList() {
      const { data: res } = await this.$http.get('goods', {
        params: this.queryInfo,
      })
      if (res.meta.status !== 200)
        return this.$message.error('获取商品列表失败')
      this.goodslist = res.data.goods
      this.total = res.data.total
      console.log(res)
    },
    //监听页码值改变
    handleCurrentChange(newPage){
        this.queryInfo.pagenum = newPage
        this.getGoodsList()
    },
    //监听条数改变
    handleSizeChange(newSize){
        this.queryInfo.pagesize = newSize
        this.getGoodsList()
    },
    //根据id删除对应的商品信息
    async removeGoodsById(id){
        console.log('id~~~',id)
        //弹框询问是否删除
        const confirmResult = await this.$confirm(
            '此操作将永久删除该商品,是否继续?',
            '提示',
            {
                confirmButtonText:'确定',
                cancelButtonText:'取消',
                type:'warning',
            }
        ).catch((err)=>{
            return err
        })
        ////promise 返回值是promise 可以用async await优化
      //如果用户确认删除，则返回值为字符串 confirm
      //如果用户取消删除，则返回值为字符串 cancel
      if(confirmResult !== 'confirm'){
          return this.$message.info('已取消删除')
      }
      //删除商品， delete
      const {data:res} = await this.$http.delete('goods/'+id)
      if(res.meta.status !== 200){
          return this.$message.error("删除商品失败")
      }
      this.$message.success("删除商品成功")
      this.getGoodsList()
    },
    goAddpage(){
        this.$router.push('/goods/add')
    }
  },
  computed: {},
}
</script>

<style lang="stylus" scoped></style>