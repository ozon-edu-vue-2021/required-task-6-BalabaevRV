<template>
  <div>
  <div class="radioGroup">
      <div>
        <input type="radio" id="static" name="typePaging" :value="true" v-model="staticPagination">
        <label for="static">Статическая</label>
      </div>

      <div>
        <input type="radio" id="infinity" name="typePaging" :value="false" v-model="staticPagination">
        <label for="infinity">Бесконечный скролл</label>
      </div>
  </div>
    <my-table 
      :rows="rows"
      :total-pages="100"
      :current-page="currentPage"
      :static-paging="staticPagination"
      @getPage="refreshPages"
    >

    <input type="text">
      <my-table-column prop="id" title="ID" />
      <my-table-column prop="postId" title="Post ID" />

      <my-table-column prop="email">
        <template #title>
          <b>Email</b>
        </template>

        <template #body="{ row }">
          <a :href="`mailto:${row.email}`">{{ row.email }}</a>
        </template>
      </my-table-column>

      <my-table-column prop="name" title="Name" />
    </my-table>
  </div>
</template>

<script>
import MyTable from './my-table';
import MyTableColumn from './my-table-column';

export default {
  name: 'BaseWrapper',
  components: {
    MyTableColumn,
    MyTable
  },
  created() {
    this.getPage(1);
  },
  watch: {
    staticPagination (newValue) {
      if (newValue) {
        this.getPage(1);    
      } else {
        this.rows = [];
        this.currentPage= 1;
        this.initPage = true;
        this.getStartPage(3);
      }

    }
  },
  data() {
    return {
      rows: [],
      currentPage: 1,
      initPage : true,
      staticPagination : true
    };
  },
  methods: {
    refreshPages (param) {
      console.log(1);
      console.log(param);
      if (this.staticPagination) {
        this.getPage(param);
      } else {
        this.infGetPage();
      }
    },
    async getPage(number) {
      const res = await fetch(`https://jsonplaceholder.typicode.com/comments?postId=${number}`);
      this.rows = await res.json();
      this.currentPage = number;
    },
    async infGetPage() {
      if (this.initPage) {
        this.initPage = false;
        return
      }
      this.blockingPromise && await this.blockingPromise;
      const res = await fetch(`https://jsonplaceholder.typicode.com/comments?postId=${this.currentPage + 1}`);
      const newRows = await res.json();
      this.rows = [...this.rows, ...newRows];
      this.currentPage++;
    },
    async getStartPage(num) {
      for (let i = 0; i <= num; i++) {
        const res = await fetch(`https://jsonplaceholder.typicode.com/comments?postId=${i}`);
        const newRows = await res.json();
        this.rows = [...this.rows, ...newRows];
      }
      this.currentPage = num;
    },
}
};
</script>

<style scoped>

  .radioGroup {
    display: flex;
  }

</style>
