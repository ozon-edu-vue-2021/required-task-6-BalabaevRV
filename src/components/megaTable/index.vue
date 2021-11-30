<template>
  <my-table 
    :rows="rows"
    :total-pages="100"
    :current-page="currentPage"
    :static-paging="false"
    @getPage="infGetPage"
  >
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
    this.getStartPage(5);
  },
  data() {
    return {
      rows: [],
      currentPage: 1,
      initPage : true
    };
  },
  methods: {
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
    async getStartPage(currentPage) {
      for (let i = 0; i <= 5; i++) {
        const res = await fetch(`https://jsonplaceholder.typicode.com/comments?postId=${i}`);
        const newRows = await res.json();
        this.rows = [...this.rows, ...newRows];
      }
      this.currentPage = currentPage;
    },
}
};
</script>
