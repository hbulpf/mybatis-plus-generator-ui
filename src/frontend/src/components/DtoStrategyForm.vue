<template>
  <el-form ref="editForm" :rules="rules" :model="form" label-width="230px">
    <el-form-item label="代码作者">
      <el-input v-model="form.author" style="width:95%;"></el-input>
    </el-form-item>
    <el-form-item prop="mapperPackage" label="Mapper的包名">
      <el-autocomplete
        v-model="form.mapperPackage"
        style="width:95%;"
        :fetch-suggestions="queryMapperPackage"
        @select="handleMapperPackageSelect"
        placeholder="例如：com.example.OrderMapper"
      >
        <el-select
          v-model="form.mapperLocationPrefix"
          style="width:110px;"
          slot="prepend"
          placeholder="请选择源码目录"
        >
          <el-option label="java" value="java"></el-option>
          <el-option label="resources" value="resources"></el-option>
        </el-select>
      </el-autocomplete>
      <help-tip
        content="生成的Mapper方法所在的Mapper.xml的包名，例如：com.example.OrderMapper，选择java或resources会在对应的源码根目录中去查找资源"
      ></help-tip>
    </el-form-item>
    <el-form-item prop="mapperMethod" label="Mapper方法名">
      <el-input
        style="width:95%;"
        v-model="form.mapperMethod"
        placeholder="例如：selectOrders"
      ></el-input>
    </el-form-item>
    <el-form-item label="开启动态SQL增强">
      <el-switch v-model="form.enableParseDynamicParams"></el-switch>
      <help-tip
        content="开启后，会自动将特定的where条件转换为mybatis的动态SQL"
      ></help-tip>
      <a href="javascript:;" @click="showDynamicParamsDemo = true">查看示例</a>
    </el-form-item>
    <el-form-item label="同时生成DTO对象及ResultMapper">
      <el-switch v-model="form.enableDTO" @change="changeEnableDTO"></el-switch>
      <help-tip
        content="开启后，可根据SQL查询结果自动生成DTO对象和ResultMapper（结果集映射表）"
      ></help-tip>
    </el-form-item>
    <el-form-item v-if="form.enableDTO" label="DTO输出位置" prop="fullPackage">
      <el-input
        style="width:95%;"
        v-model="form.fullPackage"
        placeholder="com.example.dto.ExampleDto"
      ></el-input>
      <help-tip
        content="生成文件的输出路径，包含完整的包和类名称，例如：com.example.ExampleDto"
      ></help-tip>
    </el-form-item>
    <el-form-item v-if="form.enableDTO" label="DTO是否启用Lombok注解">
      <el-switch v-model="form.enableLombok"></el-switch>
    </el-form-item>
    <el-form-item>
      <el-button type="primary" @click="onSubmit">生成代码</el-button>
    </el-form-item>
    <el-dialog
      :modal="false"
      title="动态SQL增强示例"
      :visible.sync="showDynamicParamsDemo"
      width="80%"
      top="5vh"
    >
      <dynamic-sql-demo></dynamic-sql-demo>
    </el-dialog>
  </el-form>
</template>
<script>
import axios from "axios";
import _ from "lodash";
import HelpTip from "@/components/HelpTip";
import DynamicSqlDemo from "@/components/DynamicSqlDemo";
export default {
  props: ["sql"],
  components: {
    DynamicSqlDemo,
    HelpTip,
  },
  data() {
    return {
      showDynamicParamsDemo: false,
      form: {
        enableParseDynamicParams: true,
        enableLombok: false,
        author: "",
        fullPackage: "",
        enableDTO: true,
        mapperMethod: "",
        mapperPackage: "",
        mapperLocation: "",
        mapperLocationPrefix: "resources",
      },
      rules: {
        fullPackage: [
          {
            required: true,
            message: "DTO输出位置输出位置必须填写",
            trigger: "change",
          },
        ],
        mapperPackage: [
          {
            required: true,
            message: "Mapper文件的包名必须填写",
            trigger: "change",
          },
        ],
        mapperMethod: [
          {
            required: true,
            message: "Mapper方法名必须填写",
            trigger: "change",
          },
        ],
      },
    };
  },
  mounted: function() {
    axios.get("/api/sql/basepackage").then((res) => {
      this.form.fullPackage = res + ".dto.ExampleDto";
    });
  },
  methods: {
    queryMapperPackage(queryString, cb) {
      axios
        .get("/api/ac/mapperxml", {
          params: {
            mapperLocationPrefix: this.form.mapperLocationPrefix,
            searchKey: this.form.mapperPackage,
          },
        })
        .then((res) => {
          if (res) {
            let options = res.map((v) => {
              return { value: v };
            });
            cb(options);
          }
        });
    },
    handleMapperPackageSelect(op) {
      this.form.mapperPackage = op.value;
    },
    changeEnableDTO(newVal) {
      console.log("changeEnableDTO");
      if (this.enableDTO) {
        this.rules.fullPackage = [
          {
            required: true,
            message: "DTO输出位置输出位置必须填写",
            trigger: "change",
          },
        ];
      } else {
        this.rules.fullPackage = null;
        this.form.fullPackage = "";
      }
    },
    onSubmit() {
      this.$refs["editForm"].validate((valid) => {
        if (valid) {
          this.form.mapperLocation =
            this.form.mapperPackage + "." + this.form.mapperMethod;
          axios
            .post("/api/sql/gen-mapper-method", {
              sql: this.sql,
              config: this.form,
            })
            .then((res) => {
              this.$message.success("代码已经成功生成");
              this.$emit("done");
            });
        } else {
          return false;
        }
      });
    },
  },
};
</script>
