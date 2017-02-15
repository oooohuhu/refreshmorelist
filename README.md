# refreshmorelist
上拉刷新下拉加载更多
 首先是布局 
 在EgojitPullRecyclerView中要使用到TwinklingRefreshLayout
 源码已经给出只给布局了
 <com.asuka.android.asukaandroid.widget.listViews.tkrefreshlayout.TwinklingRefreshLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/refreshLayout"
    android:orientation="vertical"
    android:layout_width="match_parent"
    app:tr_wave_height="180dp"
    app:tr_head_height="100dp"
    android:layout_height="match_parent">
    <android.support.v7.widget.RecyclerView
        android:id="@+id/recyclerview"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:overScrollMode="never" />
</com.asuka.android.asukaandroid.widget.listViews.tkrefreshlayout.TwinklingRefreshLayout>
在使用EgojitPullRecyclerView这个自定义控件
<com.asuka.android.asukaandroid.widget.listViews.tkrefreshlayout.EgojitPullRecyclerView
        android:id="@+id/listView"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="@color/colorPrimary"
        >
    </com.asuka.android.asukaandroid.widget.listViews.tkrefreshlayout.EgojitPullRecyclerView>
    
    接下来就是使用了
    
    
    
     @ViewInject(R.id.listView)
    private EgojitPullRecyclerView listView;
     //初始化
    private void initView() {
        listView.setAdapter(adapter);
        listView.getRecyclerView().setLayoutManager(new LinearLayoutManager(this,LinearLayoutManager.VERTICAL,false));
        getData(true);
    }
    
    
    //初始化事件
    private void initEvent() {
        listView.setCallBack(new RefreshCallBack() {
            @Override
            public void onRefresh() {
                pageIndex=1;

                getData(true);
            }

            @Override
            public void onLoadMore() {
                pageIndex++;
                getData(false);
            }
        });
        listView.startRefresh();
        }
        
        
       //获取数据
        private void getData(final Boolean b) {
            EGRequestParams parm=new EGRequestParams();
             parm.addBodyParameter("pageIndex",pageIndex+"");
            parm.addBodyParameter("pageSize","10");
            parm.addBodyParameter("name","");
            parm.addBodyParameter("status",status+"");
             parm.addBodyParameter("level",level+"");
            HttpHelper.post(this, UrlConfig.EventList, parm, new HttpHelper.Ok() {
                @Override
                public void success(String str) {
                    if (b){
                        list.clear();
                        list=JSONArray.parseArray(str);
                    }else {
                        list.addAll(JSONArray.parseArray(str));
                    }
                    adapter.setDataList(list);
                    listView.endRefresh();
                }

                @Override
                public void complete(String str) {

                }
            });
    }
    
    
    //adapter取数据展示
    public BaseRecyclerAdapter adapter=new BaseRecyclerAdapter() {
        @Override
        public CommonHolder setViewHolder(ViewGroup parent) {
            return new CardHolder(parent.getContext(), parent);
        }

        class CardHolder extends CommonHolder<JSONObject> {
            private TextView name;
            private View btnFenFa;
            private TextView type;
            private TextView level;
            private TextView url;
            private TextView address;
            public CardHolder(Context context, ViewGroup root) {

                super(context, root, R.layout.item_fragment1);
            }

            @Override
            public void bindData(final JSONObject obj) {
                name = (TextView) itemView.findViewById(R.id.name);
                type = (TextView) itemView.findViewById(R.id.type);
                level = (TextView) itemView.findViewById(R.id.level);
                url = (TextView) itemView.findViewById(R.id.url);
                address = (TextView) itemView.findViewById(R.id.address);
                btnFenFa=itemView.findViewById(R.id.btnFenFa);
                String eventstatus=obj.getString("eventstatus");
                String btnStr=getBtnStr(eventstatus);
                String eventType=obj.getString("eventtype");
                String eventTypeName=eventType;
                final String id=obj.getString("id");
                final String outdate=obj.getString("outdate");
              
                address.setText("地区:"+obj.getString("companyarea"));
                btnFenFa.setOnClickListener(new View.OnClickListener() {
                    @Override
                    public void onClick(View v) {
                        getCityList(id);
                    }
                });
                itemView.setOnClickListener(new View.OnClickListener() {
                    @Override
                    public void onClick(View v) {
                        Bundle bundle=new Bundle();
                        bundle.putString("id",id);
                        bundle.putString("outdate",outdate);
                        startActivity(ShiJianLieBiaoDetailActivity.class,"事件详情",bundle);
                    }
                });
            }
        }
    };
        
