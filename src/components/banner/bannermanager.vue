<template>
  <div>
    <el-row style="margin-bottom:10px;">
      <el-col :span="24">
        <div>
          <el-button type="primary" icon="el-icon-plus" @click="showAddDialog=true">添加轮播</el-button>
        </div>
      </el-col>
    </el-row>
    <el-row>
      <el-col :span="24">
        <el-table
          :data="tableData"
          style="width: 100%">
          <el-table-column
            prop="date"
            label="轮播编号"
            width="180">
          </el-table-column>
          <el-table-column
            prop="name"
            label="轮播图片"
            width="180">
          </el-table-column>
          <el-table-column
            prop="address"
            label="跳转地址">
          </el-table-column>
          <el-table-column
            fixed="right"
            label="操作"
            width="100">
            <template slot-scope="scope">
              <el-button
                size="mini"
                @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
              <el-button
                size="mini"
                type="danger"
                @click="handleDelete(scope.$index, scope.row)">删除</el-button>
            </template>
          </el-table-column>
        </el-table>
      </el-col>
    </el-row>

    <el-row style="margin-top:10px;">
      <el-button type="primary">首页</el-button>
      <el-button type="success">上一页</el-button>
      <el-button type="success">下一页</el-button>
      <el-button type="primary">尾页</el-button>
    </el-row>



    <el-dialog title="添加轮播" :visible.sync="showAddDialog">
      <el-form>
        <el-upload
          class="upload-demo"
          ref="upload"
          action=""
          :auto-upload="false">
          <el-button slot="trigger" size="small" type="primary">请选择轮播图片</el-button>
          <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过1MB</div>
        </el-upload>
        <el-form-item label="活动区域">
          <el-select placeholder="请选择活动区域" v-model="option">
            <el-option label="区域一" value="shanghai"></el-option>
            <el-option label="区域二" value="beijing"></el-option>
          </el-select>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="showAddDialog = false">取 消</el-button>
        <el-button type="primary" @click="showAddDialog = false">确 定</el-button>
      </div>
    </el-dialog>


  </div>
</template>

<script>
  export default {
    name: "bannermanager",
    data(){
      return {
        tableData:[],
        showAddDialog:false,
        option:''
      }
    },
    mounted(){

    },
    methods:{
      /**
       * 表格编辑按钮
       * @param index
       * @param row
       */
      handleEdit(index,row)
      {
        console.info(index);
        console.info(row);
      },
      /**
       * 表格删除按钮
       * @param index
       * @param row
       */
      handleDelete(index,row)
      {
        console.info(index);
        console.info(row);
      }
    }
  }
</script>

<style scoped>

</style>
