---
title: RecyclerView上拉加载最佳实践
date: 2025-07-22
---



```kotlin
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import android.widget.ProgressBar
import android.widget.TextView
import androidx.recyclerview.widget.RecyclerView

class MyAdapter(private val dataList: MutableList<DataItem?>) :
    RecyclerView.Adapter<RecyclerView.ViewHolder>() {

    private val VIEW_TYPE_ITEM = 0
    private val VIEW_TYPE_LOADING = 1

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): RecyclerView.ViewHolder {
        return if (viewType == VIEW_TYPE_ITEM) {
            val view = LayoutInflater.from(parent.context)
                .inflate(R.layout.item_list, parent, false)
            ItemViewHolder(view)
        } else {
            val view = LayoutInflater.from(parent.context)
                .inflate(R.layout.item_loading, parent, false)
            LoadingViewHolder(view)
        }
    }

    override fun onBindViewHolder(holder: RecyclerView.ViewHolder, position: Int) {
        if (holder is ItemViewHolder) {
            val dataItem = dataList[position]
            holder.textViewItem.text = dataItem?.content
        } else if (holder is LoadingViewHolder) {
            // 加载视图不需要特别的绑定逻辑，显示即可
        }
    }

    override fun getItemCount(): Int = dataList.size

    override fun getItemViewType(position: Int): Int {
        return if (dataList[position] == null) VIEW_TYPE_LOADING else VIEW_TYPE_ITEM
    }

    // 添加数据到列表
    fun addData(newData: List<DataItem>) {
        val startPosition = dataList.size
        dataList.addAll(newData)
        notifyItemRangeInserted(startPosition, newData.size)
    }

    // 显示加载更多视图
    fun showLoading() {
        if (!isLoadingVisible()) {
            dataList.add(null) // null 表示加载更多视图
            notifyItemInserted(dataList.size - 1)
        }
    }

    // 隐藏加载更多视图
    fun hideLoading() {
        if (isLoadingVisible()) {
            val lastPosition = dataList.size - 1
            dataList.removeAt(lastPosition)
            notifyItemRemoved(lastPosition)
        }
    }

    // 检查加载视图是否可见
    private fun isLoadingVisible(): Boolean {
        return dataList.isNotEmpty() && dataList.last() == null
    }

    inner class ItemViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {
        val textViewItem: TextView = itemView.findViewById(R.id.textViewItem)
    }

    inner class LoadingViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {
        val progressBar: ProgressBar = itemView.findViewById(R.id.progressBar)
        val textViewLoading: TextView = itemView.findViewById(R.id.textViewLoading)
    }
}
```

```kotlin
import android.os.Bundle
import android.os.Handler
import android.os.Looper
import android.util.Log
import androidx.appcompat.app.AppCompatActivity
import androidx.recyclerview.widget.LinearLayoutManager
import androidx.recyclerview.widget.RecyclerView

class MainActivity : AppCompatActivity() {

    private lateinit var recyclerView: RecyclerView
    private lateinit var adapter: MyAdapter
    private val dataItems = mutableListOf<DataItem?>() // 使用 DataItem? 允许 null 作为加载视图
    private var isLoading = false // **最佳实践1：标记是否正在加载数据，防止重复加载**
    private var currentPage = 1 // 当前页码
    private val ITEMS_PER_PAGE = 20 // 每页加载的数量
    private val PRELOAD_THRESHOLD = 5 // **最佳实践2：预加载阈值，距离底部还有 N 个item时开始加载**

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        recyclerView = findViewById(R.id.recyclerView)
        // 设置布局管理器
        val layoutManager = LinearLayoutManager(this)
        recyclerView.layoutManager = layoutManager

        // 初始化适配器
        adapter = MyAdapter(dataItems)
        recyclerView.adapter = adapter

        // 初始化加载第一页数据
        loadMoreData()

        // 添加滚动监听器
        recyclerView.addOnScrollListener(object : RecyclerView.OnScrollListener() {
            override fun onScrolled(recyclerView: RecyclerView, dx: Int, dy: Int) {
                super.onScrolled(recyclerView, dx, dy)

                // 获取列表总项数
                val totalItemCount = layoutManager.itemCount
                // **最佳实践3：获取最后一个可见项的索引**
                val lastVisibleItemPosition = layoutManager.findLastVisibleItemPosition()

                // 判断是否需要加载更多数据：
                // 1. dy > 0: 确保用户是向下滚动 (向上滚动不触发加载)
                // 2. !isLoading: 确保当前没有正在进行的数据加载
                // 3. lastVisibleItemPosition >= totalItemCount - 1 - PRELOAD_THRESHOLD: 判断是否达到预加载条件
                if (dy > 0 && !isLoading && lastVisibleItemPosition >= totalItemCount - 1 - PRELOAD_THRESHOLD) {
                    Log.d("MainActivity", "达到预加载阈值，开始加载更多...")
                    loadMoreData()
                }
            }
        })
    }

    /**
     * 模拟加载更多数据的方法。
     * 在实际应用中，这里会替换为真正的网络请求。
     */
    private fun loadMoreData() {
        isLoading = true // 开始加载，设置标志位为 true
        adapter.showLoading() // 显示加载更多视图

        // 模拟2秒的网络延迟加载数据
        Handler(Looper.getMainLooper()).postDelayed({
            adapter.hideLoading() // 隐藏加载更多视图

            val newItems = mutableListOf<DataItem>()
            // 模拟生成新数据
            for (i in 0 until ITEMS_PER_PAGE) {
                val id = (currentPage - 1) * ITEMS_PER_PAGE + i + 1
                newItems.add(DataItem(id, "Item $id - Page $currentPage"))
            }

            adapter.addData(newItems) // 将新数据添加到适配器中
            currentPage++ // 页码递增
            isLoading = false // 加载完成，设置标志位为 false
            Log.d("MainActivity", "数据加载完成，当前页码: $currentPage")

            // 可以在此处添加判断是否还有更多数据的逻辑
            // 例如：if (newItems.isEmpty() && currentPage > initialPage) { showNoMoreDataFooter() }
        }, 2000)
    }
}
```

