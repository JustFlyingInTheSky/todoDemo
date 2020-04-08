<template>
  <div id="app">
    <a-input
      placeholder="请输入任务"
      v-model="inputTaskValue"
      @keyup.enter="addTask"
      class="inputClass"
    />
    <a-button type="primary" @click="addTask">添加事项</a-button>

    <a-list bordered :dataSource="taskList" class="listClass">
      <a-list-item slot="renderItem" slot-scope="item">
        <!-- 复选框 -->
        <a-checkbox :checked="item.done" @change="handleChecked(item)">
          <span class="fontColor">{{item.description}}</span>
        </a-checkbox>

        <!-- 删除 -->
        <a slot="actions" @click="delTask(item.id)">删除</a>
      </a-list-item>

      <!-- footer区域 -->
      <div slot="footer" class="footer">
        <!-- 未完成的任务个数 -->
        <span>{{undoneTaskCount}}项未完成任务</span>

        <!-- 操作按钮 -->
        <a-button-group>
          <a-button
            :type="buttonChecked === 'all' ? 'primary' : 'default'"
            @click="changeButton('all')"
          >全部</a-button>
          <a-button
            :type="buttonChecked === 'undone' ? 'primary' : 'default'"
            @click="changeButton('undone')"
          >未完成</a-button>
          <a-button
            :type="buttonChecked === 'done' ? 'primary' : 'default'"
            @click="changeButton('done')"
          >已完成</a-button>
          <a-button @click="cleanDone">清除已完成</a-button>
          <a-button @click="resetAll">重置</a-button>
        </a-button-group>
      </div>
    </a-list>
  </div>
</template>

<script>
export default {
  name: "app",
  data() {
    return {
      taskList: [], //  从localStorage获取并绑定到页面的数据
      nextId: 0,  //  设置下一个任务的key
      inputTaskValue: "", //  双向绑定到input
      inputTask: {},  //  放置任务对象
      realLength: 0,  //  最大长度，用于获取localStorage的数据
      buttonChecked: "all", //  选中的按钮
      undoneTaskCount: 0  //  未完成任务数量
    };
  },

  created() {
    // 打开页面时获取并绑定数据
    if (localStorage.getItem("nextId")) {
      this.nextId = localStorage.getItem("nextId");
    } else {
      this.nextId = 0;
    }
    localStorage.setItem("nextId", this.nextId);
    this.getTasks();
  },

  methods: {
    // 更新localStorage的内容长度
    updateRealLength() {
      if (localStorage.length >= this.nextId) {
        this.realLength = localStorage.length;
      } else {
        this.realLength = this.nextId;
      }
    },

    // 添加任务
    addTask() {
      if (this.inputTaskValue.trim().length <= 0) {
        return this.$message.warning("没有填写任务");
      }
      // 构造任务对象
      this.inputTask = {
        id: this.nextId,
        description: this.inputTaskValue.trim(),
        done: false
      };
      // 存储任务到localStorage
      localStorage.setItem(this.nextId, JSON.stringify(this.inputTask));
      // 更新nextid
      this.nextId++;
      localStorage.setItem("nextId", this.nextId);
      // 清空input
      this.inputTaskValue = "";
      // 更新页面数据
      this.getTasks();
    },

    // 从localStorage获取数据
    getTasks() {
      this.updateRealLength();
      this.taskList = [];
      for (let i = 0; i < this.realLength; i++) {
        if (localStorage.getItem(i)) {
          this.taskList.unshift(JSON.parse(localStorage.getItem(i)));
        }
      }
      this.getUndoneTaskCount();
    },

    // 通过id删除localStorage中的数据
    delTask(id) {
      localStorage.removeItem(id);
      this.getTasks();
      this.changeTaskList();
    },

    // 重置清空所有数据
    resetAll() {
      this.updateRealLength();
      for (let i = 0; i < this.realLength; i++) {
        if (localStorage.getItem(i)) {
          localStorage.removeItem(i);
        }
      }

      this.taskList = [];
      this.nextId = 0;
      this.inputTaskValue = "";
      this.inputTask = {};
      this.realLength = 0;

      localStorage.setItem("nextId", this.nextId);
      this.buttonChecked = "all";
      this.getUndoneTaskCount();
      this.$message.warning("已经重置");
    },

    // 更改完成状态
    handleChecked(item) {
      item.done = !item.done;
      localStorage.setItem(item.id, JSON.stringify(item));
      this.getTasks();
      this.changeTaskList();
    },

    // 切换按钮
    changeButton(status) {
      // 更改选中的按钮
      this.buttonChecked = status;
      // 通过选中的按钮来获取需要显示的数据
      this.getTasks();
      // 筛选数据
      this.changeTaskList();
      this.getUndoneTaskCount();
    },

    // 维护未完成数量
    getUndoneTaskCount() {
      this.updateRealLength();
      this.undoneTaskCount = 0;
      for (let i = 0; i < this.realLength; i++) {
        if (
          localStorage.getItem(i) &&
          JSON.parse(localStorage.getItem(i)).done == false
        ) {
          this.undoneTaskCount++;
        }
      }
    },

    // 清空已完成任务
    cleanDone() {
      this.updateRealLength();
      for (let i = 0; i < this.realLength; i++) {
        if (
          localStorage.getItem(i) &&
          JSON.parse(localStorage.getItem(i)).done == true
        ) {
          localStorage.removeItem(i);
        }
      }
      // 切换到全部数据
      this.changeButton("all");
      this.$message.warning("成功清除已完成任务");
    },

    // 筛选当前显示的数据
    changeTaskList() {
      if (this.buttonChecked === "all") {
        return;
      } else if (this.buttonChecked === "undone") {
        this.taskList = this.taskList.filter(item => item.done === false);
      } else {
        this.taskList = this.taskList.filter(item => item.done === true);
      }
    }
  }
};
</script>

<style scoped>
#app {
  padding: 10px;
  position: absolute;
  left: 50%;
  margin-left: -360px;
}

.inputClass {
  width: 600px;
  margin-right: 10px;
}

.button_size {
  width: 90px;
}

.listClass {
  width: 700px;
  margin-top: 10px;
}

.footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

span {
  color: red;
}

.fontColor {
  color: black;
}
</style>
