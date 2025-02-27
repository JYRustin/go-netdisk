<template>

  <div>
    <div class="text-center" v-show="pager.loading">
      <i class="fa fa-spinner fa-spin fa-3x fa-fw"></i>
    </div>

    <div v-show="!pager.loading && pager.isEmpty() && !pager.errorMessage">
      <div class="italic text-center">
        {{emptyHint?emptyHint:$t('plugin.emptyHintDefault')}}
      </div>
    </div>

    <div class="cursor" v-show="pager.errorMessage" @click="refresh">

      <div class="text-center">
        <div>
          <img class="img-md" src="../assets/img/error.png"/>
        </div>
        <div class="mt10">
          {{pager.errorMessage}}
        </div>
        <div>
          {{$t('plugin.clickRefresh')}}
        </div>
      </div>
    </div>

    <div v-if="showPagination" class="text-center">
      <nav>
        <ul class="pagination mt5 mb0">

          <li v-show="!isFirstPage">
            <a href="javascript:void(0)" @click.stop.prevent="changePage(1)">&laquo;</a>
          </li>
          <li v-show="!isFirstPage">
            <a href="javascript:void(0)" @click.stop.prevent="changePage(pager.page-1)">&lsaquo;</a>
          </li>

          <li v-for="indicator in indicators" :class="{active: indicator === pager.page}">
            <a href="javascript:void(0)" @click.stop.prevent="changePage(indicator)">{{indicator}}</a>
          </li>

          <li v-show="!isLastPage">
            <a href="javascript:void(0)" @click.stop.prevent="changePage(pager.page+1)">&rsaquo;</a>
          </li>
          <li v-show="!isLastPage">
            <a href="javascript:void(0)" @click.stop.prevent="changePage(totalPages)">&raquo;</a>
          </li>
        </ul>
      </nav>

    </div>

    <div class="text-center mt10" v-show="pager.totalItems > pager.pageSize">
      {{$t('plugin.everyPage')}}
      <select v-model="pageSize">
        <option v-for="size in pageSizeOptions" v-if="size !== pager.pageSize" :value="size">{{size}}</option>

        <option :value="pager.pageSize" disabled>
          {{pager.pageSize}}
        </option>

      </select>
      {{$t('plugin.items')}}

      {{$t('plugin.total')}} {{pager.totalItems}} {{$t('plugin.items')}}
    </div>

  </div>

</template>

<script>

  import Pager from "../model/base/Pager";
  import Vue from "vue"

  export default {
    data() {
      return {
        //引入内部的pageSize变量，防止enableHistory出现两次刷新。并且强制置为page=0. -1表示未初始化。
        pageSize: -1,
        pageSizeOptions: [5, 10, 15, 20, 30, 50]
      }
    },
    props: {
      pager: {
        type: Pager,
        required: true,
        validator: function (value) {

          if (!value.offset) {
            value.offset = 3;
          }
          value.offset = parseInt(value.offset);

          return true;
        }
      },
      callback: {
        type: Function,
        required: true
      },
      emptyHint: {
        type: String,
        required: false,
        "default": ""
      }
    },
    computed: {
      showPagination() {
        return this.pager.totalItems > this.pager.pageSize;
      },
      isFirstPage() {
        return this.pager.page === 1;
      },
      totalPages() {
        return Math.ceil(this.pager.totalItems / this.pager.pageSize);
      },
      isLastPage() {
        return this.pager.page === this.totalPages
      },
      colSize() {
        return this.pager.offset * 2 + 1;
      },
      indicators() {

        let arr = [];
        //only one group. start from 1.
        if (this.totalPages <= this.colSize) {
          for (let i = 1; i < 1 + this.totalPages; i++) {
            arr.push(i);

          }
        } else {

          //many groups.
          //very close to beginning
          if ((this.pager.page + 1) * 2 < this.colSize) {
            for (let i = 1; i < 1 + this.colSize; i++) {
              arr.push(i)

            }
          } else if (this.pager.page + 1 + this.pager.offset > this.totalPages) {

            //very close to the end

            for (let i = this.totalPages - this.colSize + 1; i < 1 + this.totalPages; i++) {
              arr.push(i)

            }
          } else {

            //at the middle
            for (let i = this.pager.page - this.pager.offset + 2; i < 2 + this.pager.page + this.pager.offset; i++) {
              arr.push(i)

            }
          }
        }


        return arr;
      }

    },
    watch: {

      //场外的改变，我们只做调整，但是不刷新。
      "pager.pageSize"(newVal, oldVal) {

        if (newVal) {
          newVal = parseInt(newVal);
        } else {
          newVal = 10;
        }

        if (oldVal) {
          oldVal = parseInt(oldVal);
        } else {
          oldVal = 10;
        }

        this.pager.pageSize = newVal;

        if (newVal !== oldVal) {
          this.pageSize = this.pager.pageSize;
        }
      },
      //场内主动改变，我们必须去刷新
      "pageSize"(newVal, oldVal) {


        if (oldVal === -1) {
          return;
        }


        if (newVal) {
          newVal = parseInt(newVal);
        } else {
          newVal = 10;
        }

        if (oldVal) {
          oldVal = parseInt(oldVal);
        } else {
          oldVal = 10;
        }

        if (newVal === oldVal) {
          return;
        }

        this.pageSize = newVal;

        //保证是用户点击了select而改变的，而不是"pager.pageSize"改变了导致的。
        if (this.pager.pageSize !== this.pageSize) {
          this.pager.pageSize = this.pageSize;
          this.page = 1;
          this.refresh();
        }
      }
    },
    methods: {

      changePage(page) {
        debugger;
        if (this.pager.page !== page) {

          this.pager.page = page;
          this.refresh();
        }

      },
      refresh() {
        if (typeof this.callback === "function") {
          this.callback();
        }
      }
    },
    mounted() {
      this.pageSize = this.pager.pageSize;

      if (!this.pager.offset) {
        this.pager.offset = 3;
      }
    }
  }
</script>
