<template>
  <div class="board-list">
    <div class="common-buttons">
      <button type="button" class="w3-button w3-round w3-blue-gray" v-on:click="fnWrite">등록</button>
    </div>
    <table class="w3-table-all">
      <thead>
        <tr>
          <th>No</th>
          <th>제목</th>
          <th>작성자</th>
          <th>등록일시</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(row, id) in list" :key="id">
          <td>{{ row.id }}</td>
          <td><a v-on:click="fnView(`${row.id}`)">{{ row.title }}</a></td>
          <td>{{ row.author }}</td>
          <td>{{ row.createdAt }}</td>
        </tr>
      </tbody>
    </table>
    <div class="pagination w3-bar w3-padding-16 w3-small" v-if="paging.totalElements > 0">
      <span class="pg">
        <!-- << Button -->
        <a href="javascript:;" @click="fnPage(firstPage() - 10)" class="first w3-button w3-bar-item w3-border" :class="{ disabled: firstPage() <= 1 }">&lt;&lt;</a>
        <!-- < Button -->
        <a href="javascript:;" @click="fnPage(paging.number - 1)" class="prev w3-button w3-bar-item w3-border" :class="{ disabled: paging.number <= 0 }">&lt;</a>
        
        <!-- Page Number Buttons -->
        <template v-for="n in paginavigation()">
          <template v-if="paging.number === n">
            <strong class="w3-button w3-bar-item w3-border w3-green" :key="n">{{ n + 1 }}</strong>
          </template>
          <template v-else>
            <a class="w3-button w3-bar-item w3-border" href="javascript:;" @click="fnPage(n)" :key="n">{{ n + 1 }}</a>
          </template>
        </template>

        <!-- > Button -->
        <a href="javascript:;" @click="fnPage(paging.number + 1)" class="next w3-button w3-bar-item w3-border" :class="{ disabled: paging.number >= paging.totalPages - 1 }">&gt;</a>

        <!-- >> Button -->
        <a href="javascript:;" @click="fnPage(firstPage() + 10)" class="last w3-button w3-bar-item w3-border" :class="{ disabled: firstPage() + 10 > paging.totalPages }">&gt;&gt;</a>
      </span>
    </div>
  </div>
</template>

<script>
  export default {
    data() {
      return {
        requestBody: {},
        list: [],
        no: '',
        paging: {
          number: 0, // 현재 페이지 (0-based index)
          size: 10, // 페이지당 항목 수
          totalElements: 0, // 전체 항목 수
          totalPages: 0 // 전체 페이지 수
        },
        page: this.$route.query.page ? parseInt(this.$route.query.page) : 1,
        size: this.$route.query.size ? parseInt(this.$route.query.size) : 10,
        keyword: this.$route.query.keyword,
        paginavigation: function () {
          let pageNumber = [];
          let totalPages = this.paging.totalPages;
          let currentPage = this.paging.number + 1; // 1-based index for current page
          
          // Determine start and end page for pagination
          let startPage = Math.max(1, Math.floor((currentPage - 1) / 10) * 10 + 1);
          let endPage = Math.min(totalPages, startPage + 9);

          for (let i = startPage; i <= endPage; i++) {
            pageNumber.push(i - 1); // 0-based index for page buttons
          }

          return pageNumber;
        },
        firstPage() {
          return Math.floor(this.paging.number / 10) * 10;
        }
      }
    },
    mounted() {
      this.fnGetList()
    },
    methods: {
      fnGetList() {
        this.requestBody = {
          keyword: this.keyword,
          page: this.page - 1, // API 요청은 0-based index로 페이지를 요구할 수 있음
          size: this.size
        }
        this.$axios.get(this.$serverUrl + "/boards", {
          params: this.requestBody,
          headers: {}
        }).then((res) => {
          if (res.data != null) {
            this.list = res.data.content
            this.paging.number = res.data.number; // 현재 페이지 번호 (0-based index)
            this.paging.size = res.data.size
            this.paging.totalElements = res.data.totalElements
            this.paging.totalPages = res.data.totalPages
            this.no = this.paging.number + 1 // 1-based index 표시
          }
        }).catch((err) => {
          if (err.message.indexOf('Network Error') > -1) {
            alert('네트워크가 원활하지 않습니다.\n잠시 후 다시 시도해주세요.')
          }
        })
      },
      fnView(id) {
        this.requestBody.id = id
        this.$router.push({
          path: './detail',
          query: this.requestBody
        })
      },
      fnWrite() {
        this.$router.push({
          path: './write'
        })
      },
      fnPage(n) {
        if (n >= 0 && n < this.paging.totalPages) {
          this.page = n + 1; // 1-based index
          this.fnGetList();
        }
      }
    }
  }
</script>

<style>
  .disabled {
    pointer-events: none;
    opacity: 0.5;
  }
</style>
