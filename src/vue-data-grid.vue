<template>
  <div id="cmg-grid" class="pb-6">
    <div class="pb-4 mb-10" :class="{'border-b border-gray-200': !isMainPage}">
      <h3>{{page_title}}</h3>
    </div>
    <!-- <div v-if="$cookies.get('reservation_filters')">
      <button @click="applySaveFilters" class="cmg-btn secondary-btn mr-3">Appliquer les filtres enregistrés</button>
    </div> -->
    <!-- filters -->
    <div class="nav flex justify-end">
      <input
        v-if="searchbar"
        type="text"
        placeholder="Recherche"
        class="search-filter cmg-input cmg-input-icon cmg-input-flat mx-2.5"
        v-model="search"
        @keyup.enter="applyFilters"
      >
      <button
        v-if="filterLength"
        @click="openModal"
        class="filter-btn mx-2.5"
        :class="{'open': modal}"
      >
        <!-- <span class="icon"><Filters /></span> -->
        <span class="text">Filtrer</span>
        <span class="nbr">{{filterActive.length}}</span>
        <!-- <span class="arrow"><ArrowDown /></span> -->
      </button>
      <button class="cmg-btn primary-btn ml-2.5" v-if="new_btn" @click="btn_function">{{text_new_btn}}</button>
    </div>
    <div class="modal-wrap">
      <div v-if="modal" @click="closeModal" class="modal-closer"></div>
      <div v-show="modal" class="modal-filters">
        <!-- <div class="manage-filters">
          <button
            @click="saveFilters"
            class="cmg-btn secondary-btn mr-3"
          >
            Enregistrer les filtres
          </button>
          <button
            @click="applyFilters"
            class="cmg-btn primary-btn"
          >
            Appliquer
          </button>
        </div> -->
        <div class="filters-wrap">
          <div v-for="(col, i) in cols" :key="i" class="col">
            <div v-if="col.filter && col.filter.type === 'date'" class="col-content">
              <label :class="{'active': filterActive.includes(col.key)}">{{col.value}}</label>
              <div class="picker-wrap">
                <input
                  filter="onDate"
                  :column="col.key"
                  class="picker cmg-input"
                  type="text"
                  autocomplete="off"
                  placeholder="Date"
                >
              </div>
              <div class="picker-wrap">
                <input
                  filter="beforeDate"
                  :column="col.key"
                  class="picker cmg-input"
                  type="text"
                  autocomplete="off"
                  placeholder="< Avant"
                >
              </div>
              <div class="picker-wrap">
                <input
                  filter="afterDate"
                  :column="col.key"
                  class="picker cmg-input"
                  type="text"
                  autocomplete="off"
                  placeholder="> Après"
                >
              </div>
            </div>
            <div v-if="col.filter && col.filter.type === 'time' || col.filter && col.filter.type === 'timeDate'" class="col-content">
              <label :class="{'active': filterActive.includes(col.key)}">{{col.value}}</label>
              <input
                filter="onTime"
                :column="col.key"
                class="cmg-input"
                type="time"
                autocomplete="off"
                placeholder="Tout pile"
                v-model="col.filter.onTime"
              >
              <input
                filter="beforeTime"
                :column="col.key"
                class="cmg-input"
                type="time"
                autocomplete="off"
                placeholder="< Inférieur"
                v-model="col.filter.beforeTime"
                @blur="col.filter.onTime = ''"
              >
              <input
                filter="afterTime"
                :column="col.key"
                class="cmg-input"
                type="time"
                autocomplete="off"
                placeholder="> Supérieur"
                v-model="col.filter.afterTime"
                @blur="col.filter.onTime = ''"
              >
            </div>
            <div v-if="col.filter && col.filter.type === 'number'" class="col-content">
              <label :class="{'active': filterActive.includes(col.key)}">{{col.value}}</label>
              <input
                filter="onNumber"
                :column="col.key"
                class="cmg-input"
                type="number"
                autocomplete="off"
                placeholder="Tout pile"
                v-model="col.filter.onNumber"
              >
              <input
                filter="beforeNumber"
                :column="col.key"
                class="cmg-input"
                type="number"
                autocomplete="off"
                placeholder="< Inférieur"
                v-model="col.filter.beforeNumber"
                @blur="col.filter.onNumber = ''"
              >
              <input
                filter="afterNumber"
                :column="col.key"
                class="cmg-input"
                type="number"
                autocomplete="off"
                placeholder="> Supérieur"
                v-model="col.filter.afterNumber"
                @blur="col.filter.onNumber = ''"
              >
            </div>
            <div v-if="(col.filter && col.filter.type === 'list') || (col.filter && col.filter.type === 'multiList')" class="col-content">
              <label :class="{'active': filterActive.includes(col.key)}">{{col.value}}</label>
              <div class="cmg-multiselect">
                <div :id="`multi-open-${i}`" @click="$multiOpen(`#multi-open-${i}`)" class="input-content">
                  <!-- <Loupe /> -->
                  <span>{{col.filter.placeholder}}</span>
                  <input :column="col.key" :ref="col.key" type="text" values="">
                </div>
                <ul>
                  <li
                    v-for="(option, op) in col.filter.options"
                    :key="op"
                    :id="`multi-select-${i}-${op}`"
                    @click="$multiSelect(`#multi-select-${i}-${op}`), selectFilter(col.key)"
                    :value="option"
                    :title="option"
                  >
                    {{option}}
                  </li>
                </ul>
              </div>
            </div>
            <div v-if="col.filter && col.filter.type === 'api'" class="col-content">
              <label :class="{'active': filterActive.includes(col.key)}">{{col.value}}</label>
              <div class="cmg-multiselect">
                <div :id="`multi-open-${i}`" @click="$multiOpen(`#multi-open-${i}`)" class="input-content">
                  <!-- <Loupe /> -->
                  <span>{{col.filter.placeholder}}</span>
                  <input :column="col.key" :ref="col.key" type="text" values="">
                </div>
                <ul>
                  <li
                    v-for="(option, op) in col.filter.options"
                    :key="op"
                    :id="`multi-select-${i}-${op}`"
                    @click="$multiSelect(`#multi-select-${i}-${op}`), selectSpecialFilter(Object.values(option))"
                    :value="Object.keys(option)"
                    :title="Object.keys(option)"
                  >
                    {{Object.keys(option)[0]}}
                  </li>
                </ul>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
    <div class="table-wrap" :style="tableHeight()">
      <table>
        <thead
          :class="{
            withCheckbox: checkbox,
            sticky: stickyThead,
            sticked: scroll.top > 0
          }"
        >
          <tr :class="{ withCheckbox: checkbox }">
            <th
              v-if="checkbox"
              class="checkbox"
              :class="{ sticky: stickyCol }"
            >
              <input type="checkbox" @click="selectRow('all')" />
            </th>
            <th
              v-for="(col, i) in cols"
              :key="i"
              @click="toggleSort(col.key)"
              :class="{
                sticky: stickyCol && i == 0,
                hidden: col.hidden
              }"
            >
              {{ col.value }}
            </th>
            <th v-if="action_col" class="quick-actions">
              Actions
            </th>
          </tr>
        </thead>
        <tbody>
          <tr
            v-for="(row, i) in rows.slice(pagination[currentPage][0], pagination[currentPage][1])"
            :key="i"
            :class="{withCheckbox: checkbox, selected: selectedIds.includes(row.id)}"
          >
            <td
              v-if="checkbox"
              class="checkbox"
              :class="{'sticky': stickyCol}"
            >
              <input
                :data-id="row.id"
                class="table-checkbox"
                type="checkbox"
                @click="selectRow(row.id)"
                :checked="selectedIds.includes(row.id)"
              >
            </td>
            <slot :row="row" :setWidthColumn="setWidthColumn" :index="i" :stickyThead="stickyThead" :stickyCol="stickyCol"></slot>
          </tr>
        </tbody>
      </table>
      <div v-if="selectedIds.length > 0" class="multi-actions">
        <div class="tasks-selected">
          {{ selectedIds.length }}
        </div>
        <p class="mr-2 w-32 pb-0.5">
          {{ selectedIds.length > 1 ? "tâches sélectionnées" : "tâche sélectionnée" }}
        </p>
        <div class="icon">
          <!-- <Send /> -->
        </div>
        <div class="icon">
          <!-- <Phone /> -->
        </div>
        <div class="icon">
          <!-- <Chat /> -->
        </div>
        <div class="cross" @click="selectedIds = []">
          <!-- <Cross /> -->
        </div>
      </div>
    </div>
    <div v-if="pageNbr > 1" class="pagination">
      <span @click="changePage('prev')">
        <!-- <ArrowInSquareLeft /> -->
      </span>
      <p>Page {{currentPage + 1}} / {{pageNbr}}</p>
      <span @click="changePage('next')">
        <!-- <ArrowInSquareRight /> -->
      </span>
    </div>
  </div>
</template>

<script>
import $ from 'jquery'

// import Filters from '~/assets/img/filter.svg?inline'
// import ArrowDown from '~/assets/img/arrow-down.svg?inline'
// import Loupe from '~/assets/img/loupe.svg?inline'
// import Send from '~/assets/img/toolbar/send.svg?inline'
// import Phone from '~/assets/img/toolbar/phone.svg?inline'
// import Chat from '~/assets/img/toolbar/chat.svg?inline'
// import Cross from '~/assets/img/toolbar/cross.svg?inline'
// import ArrowInSquareLeft from '~/assets/img/arrow-in-square-left.svg?inline'
// import ArrowInSquareRight from '~/assets/img/arrow-in-square-right.svg?inline'


export default {
  // components: {
  //   Filters,
  //   ArrowDown,
  //   Loupe,
  //   Send,
  //   Phone,
  //   Chat,
  //   Cross,
  //   ArrowInSquareLeft,
  //   ArrowInSquareRight
  // },
  props: ['datas', 'columns', 'page_limit', 'checkbox', 'stickyThead', 'stickyCol', 'action_col', 'page_title', 'new_btn', 'text_new_btn', 'btn_function', 'searchbar', 'specialFilters', 'isMainPage'],
  data() {
    return {
      allSelected: false,
      sort: '',
      search: '',
      list: [],
      cols: this.columns,
      currentPage: 0,
      modal: false,
      selectedIds: [],
      filterActive: [],
      scroll: {
        top: 0,
        left: 0
      },
      tooltipModal: {
        top: 0,
        left: 0,
        content: null
      },
      newFetchParams: ''
    }
  },
  computed: {
    rows() {
      const sort = this.sort
      this.list.sort((a, b) => {
        let keyA, keyB
        if (typeof a[sort] === 'string' && a[sort] !== null && b[sort] !== null) {
          keyA = a[sort].toLowerCase()
          keyB = b[sort].toLowerCase()
        } else {
          keyA = a[sort]
          keyB = b[sort]
        }
        if (keyA > keyB) return -1
        if (keyA < keyB) return 1
        return 0
      })

      for (let col of this.cols) {
        if (col.key === sort && !col.sort) {
          this.list.reverse();
        }
      }

      return this.list.filter(row => {
        return Object.keys(row).some(key => {
          return (
            String(row[key])
              .toLowerCase()
              .indexOf(this.search.toLowerCase()) > -1
          )
        })
      })
    },
    pagination() {
      const pages = []
      for (let i = 1; i <= this.pageNbr; i++) {
        pages.push([(i * this.page_limit - this.page_limit), (i * this.page_limit)])
      }
      return pages
    },
    pageNbr() {
      const totalRows = this.rows.length
      if (totalRows <= this.page_limit) {
        return 1
      }
      return Math.ceil(totalRows / this.page_limit)
    },
    filterLength() {
      const filters = []
      for (let col of this.columns) {
        if (col.filter) {
          filters.push(col.filter)
        }
      }
      return filters.length
    }
  },
  methods: {
    page(row) {
      for (let i = 0; i < this.pagination.length; i++) {
        if (row >= this.pagination[i][0] && row <= this.pagination[i][1]) {
          return i
        }
      }
    },
    changePage(dir) {
      if (dir === 'prev') {
        if (this.currentPage === 0) {
          return false
        } else {
          this.currentPage--
        }
      }
      if (dir === 'next') {
        if (this.currentPage === this.pageNbr) {
          return false
        } else {
          this.currentPage++
        }
      }
    },
    selectRow(id) {
      this.selectedIds = []
      if (id === 'all') {
        this.allSelected = !this.allSelected
        if (this.allSelected) {
          this.selectedIds = this.rows.map(row => row.id)
        }
      } else {
        [].forEach.call(document.querySelectorAll('.table-checkbox'), el => {
          if (el.checked) {
            this.selectedIds.push(parseInt(el.dataset.id))
          }
        })
      }
    },
    setWidthThead() {
      for (let col = 0; col < $('#cmg-grid tbody tr:eq(0) td').length; col++) {
        const width = $(`#cmg-grid tbody tr:eq(0) td:eq(${col})`).css('min-width')
        $(`#cmg-grid thead th:eq(${col})`).css({
          'min-width': `${width}`,
          'max-width': `${width}`,
        })
      }
    },
    tableHeight() {
      let rowsNbr
      if (this.pagination.length > 0) {
        rowsNbr = this.list.length - this.pagination[this.currentPage][0] + 1
      } else {
        rowsNbr = 0
      }

      if (rowsNbr < this.page_limit) {
        // 33px = hauteur cellule
        return `height:${(rowsNbr * 33 + 33)}px`
      } else {
        return 'flex: 1'
      }
    },
    setModalContent(ref, content) {
      //:ref="`tooltip${i}`"
      //@mouseover="setModalContent(`tooltip${i}`, reservation.notes)"
      //@mouseleave="setModalContent('remove')"
      if (ref === 'remove') {
        $('.cmg-tooltip-modal').remove()
      } else {
        const top = this.$refs[ref][0].getBoundingClientRect().top
        const left = this.$refs[ref][0].getBoundingClientRect().left
        const right = this.$refs[ref][0].getBoundingClientRect().right
        const bottom = this.$refs[ref][0].getBoundingClientRect().bottom

        const x = left + (right - left)
        $(`<div class="cmg-tooltip-modal" style="top: ${top}px; left: ${x}px;">${content}</div>`).appendTo('body')
      }
    },
    setWidthColumn(width) {
      return `min-width:${width}; max-width:${width};`
    },
    toggleSort(type) {
      this.sort = type
      for (let col of this.cols) {
        if (col.key === type) {
          col.sort = !col.sort
        }
      }
    },
    openModal() {
      this.modal = true
    },
    closeModal() {
      this.modal = false
    },
    selectFilter(ref) {
      let arrayValues
      if (this.$refs[ref][0].attributes.values.nodeValue) {
        arrayValues = this.$refs[ref][0].attributes.values.nodeValue.split(',')
      } else {
        arrayValues = []
      }

      for (let col of this.cols) {
        if (col.key === ref) {
          col.filter.values = arrayValues
        }
      }
    },
    selectSpecialFilter(params) {
      this.newFetchParams += params
    },
    applyFilters() {
      if (this.newFetchParams) {
        window.location = `/reservation?${this.newFetchParams}`
      }

      this.list = this.datas
      this.filterActive = []

      for (let col of this.cols) {
        if (col.filter && col.filter.type === 'date') {
          if (col.filter.beforeDate || col.filter.afterDate) {
            this.list = this.list.filter(row => row[col.key].slice(0, 10) <= col.filter.beforeDate && row[col.key].slice(0, 10) >= col.filter.afterDate)
            this.filterActive.push(col.key)
          }
          if (col.filter.onDate) {
            this.list = this.list.filter(row => row[col.key].slice(0, 10) === col.filter.onDate)
            this.filterActive.push(col.key)
          }
        }
        if (col.filter && col.filter.type === 'time') {
          if (col.filter.beforeTime || col.filter.afterTime) {
            this.list = this.list.filter(row => row[col.key] <= col.filter.beforeTime && row[col.key] >= col.filter.afterTime)
            this.filterActive.push(col.key)
          }
          if (col.filter.onTime) {
            this.list = this.list.filter(row => row[col.key] === col.filter.onTime)
            this.filterActive.push(col.key)
          }
        }
        if (col.filter && col.filter.type === 'timeDate') {
          if (col.filter.beforeTime || col.filter.afterTime) {
            this.list = this.list.filter(row => row[col.key].slice(11, 16) <= col.filter.beforeTime && row[col.key].slice(11, 16) >= col.filter.afterTime)
            this.filterActive.push(col.key)
          }
          if (col.filter.onTime) {
            this.list = this.list.filter(row => row[col.key].slice(11, 16) === col.filter.onTime)
            this.filterActive.push(col.key)
          }
        }
        if (col.filter && col.filter.type === 'number') {
          if (col.filter.beforeNumber || col.filter.afterNumber) {
            this.list = this.list.filter(row => row[col.key] <= col.filter.beforeNumber && row[col.key] >= col.filter.afterNumber)
            this.filterActive.push(col.key)
          }
          if (col.filter.onNumber) {
            this.list = this.list.filter(row => row[col.key] === col.filter.onNumber)
            this.filterActive.push(col.key)
          }
        }
        if (col.filter && col.filter.type === 'list') {
          if (col.filter.values.length > 0) {
            this.list = this.list.filter(row => col.filter.values.includes(row[col.key]))
            this.filterActive.push(col.key)
          }
        }
        if (col.filter && col.filter.type === 'multiList') {
          if (col.filter.values.length > 0) {
            this.list = this.list.filter(row => col.filter.values.some(val => row[col.key].includes(val)))
            this.filterActive.push(col.key)
          }
        }
      }

      this.modal = false
    },
    // saveFilters() {
    //   const saveCols = []
    //   for (let col of this.cols) {
    //     if (col.filter.values) {
    //       saveCols.push({key: col.key, filter: {values: col.filter.values}})
    //     }
    //   }
    //   this.$cookies.set('reservation_filters', saveCols);
    //   alert('Filtres enregistrés')
    // },
    // applySaveFilters() {
    //   const saveCols = this.$cookies.get('reservation_filters')

    //   for (let i = 0; i < this.cols.length; i++) {
    //     for (let j = 0; j < saveCols.length; j++) {
    //       if (this.cols[i].key === saveCols[j].key) {
    //         this.cols[i].filter.values = saveCols[j].filter.values
    //       }
    //     }
    //   }

    //   this.applyFilters()
    // }
  },
  beforeMount() {
    this.list = this.datas
    for (let col of this.cols) {
      if (col.sort) {
        this.sort = col.key
      }
    }
  },
  mounted() {
    if (this.sticky) {
      const table = document.querySelector('table')
      document.querySelector('table').addEventListener('scroll', () => {
        this.scroll.top = table.scrollTop;
        // this.scroll.left = table.scrollLeft;
      })
    }

    setTimeout(() => {
      const pickerInput = document.getElementsByClassName('picker');
      const pickerDropdown = document.getElementsByClassName('litepicker');
      const pickerWrap = document.getElementsByClassName('picker-wrap');

      [].forEach.call(pickerInput, el => {
        const picker = new Litepicker({
          element: el,
          format: 'DD-MM-YYYY',
          lang: "fr-FR",
          resetButton: true,
          singleMode: true,
          numberOfcols: 1,
          numberOfMonths: 1,
          tooltipText: {
            one: 'nuit',
            other: 'nuits'
          },
          setup: (picker) => {
            const filter = el.getAttribute('filter')
            const column = el.getAttribute('column')
            picker.on('selected', (date) => {
              if (filter === 'onDate') {
                for (let col of this.cols) {
                  if (col.key === column) {
                    col.filter.onDate = this.$formatDate(date.dateInstance)
                  }
                }
              }
              if (filter === 'beforeDate') {
                for (let col of this.cols) {
                  if (col.key === column) {
                    col.filter.onDate = ''
                    $(el).closest('.col-content').find('input:eq(0)').val('')
                    col.filter.beforeDate = this.$formatDate(date.dateInstance)
                  }
                }
              }
              if (filter === 'afterDate') {
                for (let col of this.cols) {
                  if (col.key === column) {
                    col.filter.onDate = ''
                    $(el).closest('.col-content').find('input:eq(0)').val('')
                    col.filter.afterDate = this.$formatDate(date.dateInstance)
                  }
                }
              }
            });
            picker.on('clear:selection', (date) => {
              for (let col of this.cols) {
                if (col.key === column) {
                  col.filter.onDate = ''
                  col.filter.beforeDate = ''
                  col.filter.afterDate = ''
                }
              }
            });
          },
        })
      })

      for (let i = 0; i < pickerInput.length; i++) {
        pickerWrap[i].appendChild(pickerDropdown[i])
      }
    }, 1000);

    this.setWidthThead()

  },
};
</script>
