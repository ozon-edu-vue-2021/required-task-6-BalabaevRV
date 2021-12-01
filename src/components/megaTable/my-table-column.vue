<script>
export default {
  name: 'my-table-column',
  props: {
    prop: {
      type: String,
      required: true
    },
    title: {
      type: String,
      default: ''
    }
  }
};
</script>
