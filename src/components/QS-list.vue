<template>
  <div class="qs-list">
    <ul v-if="qsList.length == 0 ? false : true">
      <li></li>
      <li>标题</li>
      <li>截止时间</li>
      <li>状态</li>
      <li>操作<span @click="$router.push({name: 'qsEdit', params: {num: 0}})">+新建问卷</span></li>
    </ul>
    <template v-for="item in qsList">
      <ul>
        <li><input type="checkbox" v-model="item.checked"></li>
        <li>{{item.title}}</li>
        <li>{{item.time}}</li>
        <li :class="item.state === 'inissue' ? 'inissue' : ''">{{item.stateTitle}}</li>
        <li>
          <button @click="iterator = edit(item); iterator.next()">编辑</button>
          <button @click="iterator = delItem(item.num); iterator.next()">删除</button>
          <router-link :to="`/fill/${item.num}`" tag="button">查看问卷</router-link>
          <button @click="iterator = watchData(item); iterator.next()">查看数据</button>
        </li>
      </ul>
    </template>
    <div class="list-bottom" v-if="qsList.length == 0 ? false : true">
      <label><input type="checkbox" id="all-check" v-model="selectAll">全选</label>
      <button @click="iterator = delItems(); iterator.next()">删除</button>
    </div>
      <div class="add-qs" v-if="qsList.length === 0">
        <button class="add-btn" 
        @click="$router.push({name: 'qsEdit', params: {num: 0}})">+&nbsp;&nbsp;新建问卷</button>
      </div>
    <div class="shadow" v-if="showDialog">
      <div class="del-dialog">
        <header>
          <span>提示</span>
          <span class="close-btn" @click="showDialog = false">X</span>
        </header>
        <p>{{info}}</p>
        <div class="btn-box">
          <button class="yes" @click="iterator.next();">确定</button>
          <button @click="showDialog = false">取消</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import storage from '../store.js'

/**
 * A module that define qs-list router view
 * @exports qs-list
 * @author oyh(Reusjs)
 */
  export default {
    name: 'qsList',
    data() {
      return {
        qsList: [],
        showDialog: false,
        iterator: {},
        info: ''
      }
    },
    mounted() {
      if (storage.get() !== null) {
        this.qsList = storage.get();
        this.qsList.forEach( item => {
          let [year, month, day] = item.time.split('-')
          if (year < new Date().getFullYear()) {
            item.state = 'issueed'
            item.stateTitle = '已发布'
          } else if (year == new Date().getFullYear() 
            && month < new Date().getMonth() + 1) {
            item.state = 'issueed'
            item.stateTitle = '已发布'
          } else if (year == new Date().getFullYear() 
            && month == new Date().getMonth() + 1 
            && day < new Date().getDate()) {
            item.state = 'issueed'
            item.stateTitle = '已发布'
          }
        })
      } else {
        storage.save([

          { 'num': 1, 
            'title': '第一份问卷', 
            'time': '2030-1-1', 
            'state': 'inissue', 
            'stateTitle': '发布中', 
            'checked': false, 
            'question': [
              {'num': 'Q1', 'title': '单选题', 'type': 'radio', 'isNeed': true, 'options': ['选项一', '选项二']},
              {'num': 'Q2', 'title': '多选题', 'type': 'checkbox', 'isNeed': true, 'options': ['选项一', '选项二', '选项三', '选项四']},
              {'num': 'Q3', 'title': '文本题', 'type': 'textarea', 'isNeed': true}
            ]
          },

          { 'num': 2,
            'title': '第二份问卷',
            'time': '2030-1-1',
            'state': 'noissue',
            'stateTitle': '未发布',
            'checked': false, 
            'question': [
              {'num': 'Q1', 'title': '单选题', 'type': 'radio', 'isNeed': true, 'options': ['选项一', '选项二']},
              {'num': 'Q2', 'title': '多选题', 'type': 'checkbox', 'isNeed': true, 'options': ['选项一', '选项二', '选项三', '选项四']},
              {'num': 'Q3', 'title': '文本题', 'type': 'textarea', 'isNeed': true}
            ]
          },

          { 'num': 3,
            'title': '第三份问卷', 
            'time': '2017-3-27', 
            'state': 'issueed', 
            'stateTitle': '已发布', 
            'checked': false, 
            'question': [
              {'num': 'Q1', 'title': '单选题', 'type': 'radio', 'isNeed': true, 'options': ['选项一', '选项二']},
              {'num': 'Q2', 'title': '多选题', 'type': 'checkbox', 'isNeed': true, 'options': ['选项一', '选项二', '选项三', '选项四']},
              {'num': 'Q3', 'title': '文本题', 'type': 'textarea', 'isNeed': true}
            ]
          }
          
        ]);
        this.qsList = storage.get();
      }
    },
    methods: {
      showDialogMsg(info) {
        this.showDialog = true;
        this.info = info;
      },
      *delItem(num) {
        yield this.showDialogMsg('确认要删除此问卷')

        yield (() => {
          let index = 0;
          for (let length = this.qsList.length; index < length; index++) {
            if (this.qsList[index].num === num) break;
          }
          this.qsList.splice(index, 1);
          this.showDialog = false;
        })();
      },
      *delItems() {
        yield this.showDialogMsg('确认要删除选中的问卷？');

        yield (() => {
          this.showDialog = false;
          if (this.selectAll) {
            this.qsList = [];
            return;
          }
          if (this.selectGroup == []) return;

          this.selectGroup.forEach( item => {
            if (this.qsList.indexOf(item) > -1) this.qsList.splice(this.qsList.indexOf(item), 1);
          } )
        })();     
      },
      *edit(item) {
        yield (() => {
          if (item.state === 'noissue') {
            this.showDialogMsg('确认要编辑？');
          } else {
            this.showDialogMsg('只有未发布的问卷才能编辑');
          }
        })();
        yield (() => {
          if (item.state !== 'noissue') {
            this.showDialog = false;
          } else {
            this.showDialog = false;
            this.$router.push({name: 'qsEdit', params: { num: item.num }})
          }
        })();
      },
      *watchData(item) {
        yield (() => {
          if (item.state === 'noissue') {
            this.showDialogMsg('未发布的问卷无数据可查看');
          } else {
            this.$router.push({ name: 'qsData', params: { num: item.num }})
          }
        })();
        yield this.showDialog = false;
      }
    },
    computed: {
      selectAll: {
        get() {
          return this.selectCount === this.qsList.length && this.selectCount !== 0;
        },
        set(value) {
          this.qsList.forEach( item => {
            item.checked = value;
          } );
          return value;
        }
      },
      selectCount() {
        let i = 0;
        this.qsList.forEach( item => {
          if (item.checked) i++;
        } );
        return i;
      },
      selectGroup() {
        let group = [];
        this.qsList.forEach( item => {
          if (item.checked) group.push(item);
        } );
        return group;
      }
    },
    watch: {
      qsList: {
        handler(val) {
          val.forEach( (item, index) => {
            item.num = index + 1
          } )
          storage.save(val);
        },
        deep: true
      }
    }
  }
</script>

<style lang="scss" scoped>
@import '../style/QS-list';
</style>