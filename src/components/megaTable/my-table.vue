<script lang="jsx">
import OzTablePaginator from './oz-table-paginator';
import DotsLoaderIcon from './dost-loader.svg';

export default {
  name: 'my-table',
  props: {
    rows: {
      type: Array,
      default: () => []
    },
    totalPages: {
      type: Number,
      default: 0
    },
    currentPage: {
      type: Number,
      default: 0
    },
    staticPaging: {
      type: Boolean,
      default: true
    }    
  },
  data() {
    return {
      sortProp: '',
      sortKeys: [],
      sortProps: {},
      sortDirection: '',
      filterProp: '',
      filterText: '',
      filterArray: {
        email: "",
        name: "",
        id: {
          min: 0,
          max: 0
        }
      }
    };
  },
  computed: {
    sortedRows() {
      let res;

      res =  this.rows;

      for (let key in this.filterArray) {
        if (this.filterArray[key]) {
          if (key === 'id') {
            res = res.filter(row => row[key] >= ((this.filterArray[key].min>0) ? this.filterArray[key].min : true) && ((this.filterArray[key].max>0) ? row[key] <= this.filterArray[key].max  : true))     
          } else {
            res = res.filter(row => row[key].toLowerCase().search(this.filterArray[key].toLowerCase()) > -1)  
          }
        }
      }
      res.sort((a,b) => this.sortByMultipleKey(a,b, this.sortKeys));

      return res;
    }
  },
  methods: {
    sortByMultipleKey(a, b, keys) {
        if (keys.length === 0) return 0;
        const key = keys[0]; 
        console.log(key);
        if (this.sortProps[key] === "asc") {
          if (a[key] < b[key]) return -1; 
          else if (a[key] > b[key]) return 1; 
          else if (a[key] === b[key]) {
           if (keys.length>0) { 
              return this.sortByMultipleKey(a, b, keys.slice(1));  
           } else {
             return 0;
           }
          }       
        } else {
          if (a[key] < b[key]) return 1; 
          else if (a[key] > b[key]) return -1; 
          else if (a[key] === b[key]) {
           if (keys.length>0) { 
            return this.sortByMultipleKey(a, b, keys.slice(1));  
           } else {
             return 0;
           }
          }             
        }
    },   
    toggleSort(prop) {
      this.sortProp = prop;
      this.sortDirection = (this.sortDirection === 'desc' || !this.sortDirection) ? 'asc' : 'desc';
    },
    changeSortProps(prop) {
      if (this.sortProps[prop] === "asc") {
        this.sortProps[prop] = "desc";
      } else if (this.sortProps[prop] === "desc") {
        delete this.sortProps[prop]; 
      } else {
      this.sortProps[prop] = "asc";  
      }
      this.sortKeys = Object.keys(this.sortProps);
    },
    renderHead(h, columnsOptions) {
      const { $style } = this;
      return columnsOptions.map((column) => {
        const renderedTitle = column.scopedSlots.title ? column.scopedSlots.title() : column.title;
        let sortIcon = 'sort';
        if (this.sortProps[column.prop]) {
          sortIcon = this.sortProps[column.prop] === 'asc' ? 'sort-amount-down' : 'sort-amount-up';
        }

        return (
          <th key={column.prop} class={$style.headerCell}>
            <div class={$style.headerCellContent}>
              <span>{renderedTitle}</span>
              <font-awesome-icon
                class={$style.sortIcon}
                icon={sortIcon}
                on={{ click: () => this.changeSortProps(column.prop) }}       
              />
            </div>
          </th>
        );
      });
    },

    renderRows(h, columnsOptions) {
      return this.sortedRows.map((row, index) => {
        return <tr key={row.id || index}>{...this.renderColumns(h, row, columnsOptions)}</tr>;
      });
    },
    renderColumns(h, row, columnsOptions) {
      return columnsOptions.map((column) => {
        return (
          <td key={column.prop} class={this.$style.cell}>
            {column.scopedSlots.body ? column.scopedSlots.body({ row }) : row[column.prop]}
          </td>
        );
      });
    },
    getColumnOptions() {
      return this.$slots.default.
        filter(item => item.componentOptions && item.componentOptions.tag === 'my-table-column').
        map(column =>
          Object.assign({}, column.componentOptions.propsData, {
              scopedSlots: column.data.scopedSlots || {}
            }
          )
        );
    },
    renderInfPager() {
      const directives = [
        {
          name: 'detect-viewport',
          value: {
            callback: this.$listeners.getPage
          }
        }
      ];

      const style = {
        background: `url("${DotsLoaderIcon}") no-repeat center`
      };

      return (
        <div {...{ class: this.$style.infPager, style, directives }} />
      );
    }    
  },
  render(h) {
    const { $style, totalPages, currentPage, staticPaging, $listeners, filterArray } = this;
    const { getPage } = $listeners;
    const columnsOptions = this.getColumnOptions();
    const columnsHead = this.renderHead(h, columnsOptions);
    const rows = this.renderRows(h, columnsOptions);

    return (
      <div>
        <div class="container">
          <div>
            <h3>Фильтры</h3>  
              <div class="filter__group">
                  <label class="filter__name" for="email">E-mail</label>
                  <input type="text" id="email" name="email"
                    on={{
                      change: (e) => filterArray.email = e.target.value,
                    }}
                  />
              </div>
            <div class="filter__group">
                <label class="filter__name" for="name">Name</label>
                <input type="text" id="name" name="name" 
                  on={{
                    change: (e) => filterArray.name = e.target.value,
                  }}
                />
            </div>    
            <div class="filter__interval">
              <div class="filter__group">
                <label class="filter__name" for="idMin">id min</label>
                <input type="number" id="idMin" name="idMin" min="0"
                  on={{
                    change: (e) => filterArray.id.min = +e.target.value,
                  }}
                />
              </div> 
              <span class="filter__divider"> — </span>
              <div class="filter__group">   
                <label class="filter__name" for="idMax">id max</label>
                <input type="number" id="idMax" name="idMax" min="0"
                  on={{
                    change: (e) => filterArray.id.max = +e.target.value,
                  }}
                />  
              </div>              
            </div>                    
          </div>
          <table class={$style.table}>
            <thead>{...columnsHead}</thead>
            <tbody>{...rows}</tbody>
          </table>
        </div>  

        {staticPaging
          ? <OzTablePaginator totalPages={totalPages} currentPage={currentPage} on={{ getPage: getPage }} />
          : this.renderInfPager()
        }

      </div>
    );
  }
};
</script>

<style scoped>
  .container {
    display: flex;
  }

  .filter__group {
    display: flex;
    flex-direction: column;
    align-items: flex-start;
    margin-bottom: 20px;
  }
  
  .filter__divider {
    margin-left: 10px;
    margin-right: 10px;
  }

  .filter__interval {
    display: flex;
    align-items: center;
  }
</style>

<style module>
  .table {
    border-spacing: 0;
    margin: 8px;
    width: 100%;
  }

  .cell {
    text-align: left;
    border-bottom: 1px solid #c8cacc;
    padding: 1rem 1rem;
  }

  .headerCell {
    composes: cell;
    background: #c7cbcb;
  }

  .headerCellContent {
    display: flex;
    align-items: center;
  }

  .sortIcon {
    margin-left: 8px;
    margin-right: 24px;
  }

  .sortIcon:hover {
    cursor: pointer;
  }
</style>
