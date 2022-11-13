<template>
    <a-page-header
        style="border: 1px solid rgb(235, 237, 240)"
        title="Orders"
        sub-title="Order page"
        @back="() => $router.push('/')"
    />
    <a-row>
      <a-col :span="20" :offset="2"> 
        <div class="search-bar">
          <a-row>
            <a-col :span="4" class="search-title">
              <search-outlined /> <strong>Search</strong> 
            </a-col>
            <a-col :span="20">
              <a-input @pressEnter="handleSearchInputChange" 
                v-model:value="searchInput" 
                allow-clear 
                placeholder="Type Product Name or Order and hit enter" 
              />

            </a-col>
          </a-row>

          
        </div>

        <div class="date-range-container">
          <a-range-picker v-model:value="filterDateRange" @change="handleDateChange" />
        </div>


        <div> Total: <strong> ${{totalAmount.toFixed(2)}} </strong> </div>
        
        <a-table
            :columns="columns"
            :row-key="record => record.id"
            :data-source="dataSource"
            :pagination="false"
            :loading="loading"
            @change="handleTableChange"
        >
          <template #bodyCell="{ column, text, record }">
            <template v-if="column.dataIndex === 'order_name'">
              <div>
                {{record.order_name}} <br/>
                {{record.product_name}}
              </div>
            </template>
          </template>
        </a-table>
        <br>
        <a-pagination
          style="text-align:center"
          :current="page"
          :pageSize="pageSize"
          :pageSizeOptions="pageSizeOptions"
          show-size-changer
          show-quick-jumper
          :total="total"
          responsive
          @change="onShowSizeChange"
        />
        </a-col>
        
    </a-row>
</template>

<script setup>
  import { usePagination } from 'vue-request';
  import { computed, defineComponent , reactive ,ref,onMounted ,watch } from 'vue';
  import axios from 'axios';
  import moment from 'moment-timezone';
  import { SearchOutlined } from '@ant-design/icons-vue';
import { propsToAttrMap } from '@vue/shared';

  axios.defaults.headers.post['Content-Type'] ='application/json;charset=utf-8';
  axios.defaults.headers.post['Access-Control-Allow-Origin'] = '*';
  const columns= [
    {
      title: 'Order Name',
      dataIndex: 'order_name',
      key: 'order_name',
    },
    {
      title: 'Customer Company',
      dataIndex: 'customer_company',
      key: 'customer_company',
    },
    {
      title: 'Customer Name',
      dataIndex: 'customer_name',
      key: 'customer_name',
    },
    {
      title: 'Order Date',
      dataIndex: 'order_date',
      key: 'order_date',
      sorter: true,
      customRender: ({text,record,index})=>{
        let dateStr = record.order_date
        
        return moment.tz(dateStr,"Australia/Melbourne").format("MMM Do YYYY hA")
      }
    },
    {
      title: 'Delivered Amount',
      dataIndex: 'delivered_amount',
      key: 'delivered_amount',
      customRender: ({text,record,index})=>{
        if(record.delivered_amount == 0) return "-"
        return `$${record.delivered_amount.toFixed(2)}`
      }
    },
    {
      title: 'Total Amount',
      dataIndex: 'total_amount',
      key: 'total_amount',
      customRender: ({text,record,index})=>{
        return `$${record.total_amount.toFixed(2)}`
      }
    },
  ];

  const baseUrl = "http://localhost:9100/v1"
  const pageSizeOptions =[5,10,20,30,40,50,100]
  const totalAmount = ref(0)
  const dataSource = ref([])
  const loading = ref(false)
  const page = ref(1)
  const pageSize = ref(5)
  const total = ref(0)
  const searchInput = ref("")
  const filterDateRange = ref() 
  const params = ref({
    page: page.value,
    page_size: pageSize.value,
    search_query: ""
  })

  const pagination = computed(() => ({
    total: total.value,
    current: page.value,
    pageSize: pageSize.value,
  }));

  // First time load
  onMounted(() => {
    queryData()
  })


  // Watch search input
  watch(searchInput,() => {
    if (searchInput.value.length > 0){
      params.value.search_query = searchInput.value

    }else{
      params.value.search_query = ""
      queryData()
    }
  })


  // Fetch 
  const queryData =()=>{
    // start loading 
    loading.value = true;

    //fetch results 
    axios.get(`${baseUrl}/orders/`, {
      params : params.value,
    }).then(res=>{
      let data = res.data.data || []
      dataSource.value = data
      total.value = res.data.total
      updateTotalAmount(data)

      // end loading
      loading.value = false
    });
  }


  const updateTotalAmount =(orders)=>{
    // set total amount to zero 
    totalAmount.value = 0
    orders.forEach(order => {
      totalAmount.value += order.total_amount
    });
  }

  const handleSearchInputChange =(e)=>{
    // query on enter search input
    queryData()

  }

  const handleTableChange = (pag, filters, sorter) => {
  
    if (sorter.field?.length && sorter.order?.length){
      params.value.sort_by= `orders.created_at ${sorter.order == "descend" ? "DESC" : "ASC"  }`
    }
    else{
      params.value.sort_by= ""
    }
    params.value.filters = filters

    console.log(params)
    queryData()
  }

  const onShowSizeChange = (current_page, page_size) => {

    params.value.page_size= page_size
    params.value.page= current_page
   
    page.value = current_page
    pageSize.value = page_size
    queryData()
  };


  const handleDateChange =(timeobj,DateStr)=>{

    
    if (DateStr[0]?.length>0 && DateStr[1]?.length > 0){
      params.value.start_time = moment(DateStr[0]).format('YYYY-MM-DD')
      params.value.end_time = moment(DateStr[1]).format('YYYY-MM-DD')
    
    }
    else{
      params.value.start_time = ''
      params.value.end_time = ''
    }
    queryData()
  }




</script>


<style>
.search-bar, .date-range-container{
  margin: 20px 0px;
}

.search-title{
  font-size: 18px;
}

</style>