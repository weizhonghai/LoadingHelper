## 简介

- 深度解耦加载中、加载成功、加载失败、无数据的视图，可拓展自定义视图
- 无需在 xml 文件增加代码
- 可用于 Activity、Fragment、列表或指定的 View
- 可动态管理标题栏
- 可动态更新视图样式
- 可为视图动态增加功能方法
- 可结合大部分第三方控件使用

## 使用

在 project 的 build.gradle 添加以下代码

```
allprojects {
  repositories {
    ...
    maven { url 'https://www.jitpack.io' }
  }
}
```

在 module 的 build.gradle 添加依赖

```
dependencies {
  implementation 'com.github.CaiShenglang:LoadingHelper:1.0.0-alpha'
}
```

### 基础用法

继承 LoadingHelper.Adapter&lt;VH extends ViewHolder&gt;，用法与 RecyclerView.Adapter 类似

```
public class LoadingAdapter extends LoadingHelper.Adapter<LoadingHelper.ViewHolder> {
  
  @NonNull
  @Override
  public LoadingHelper.ViewHolder onCreateViewHolder(@NonNull LayoutInflater inflater, @NonNull ViewGroup parent) {
    return new LoadingHelper.ViewHolder(inflater.inflate(R.layout.lce_layout_loading_view, parent, false));
  }

  @Override
  public void onBindViewHolder(@NonNull LoadingHelper.ViewHolder holder) {

  }
}
```

调用 registerAdapter(int viewType, LoadingHelper.Adapter adapter) 注册对应类型的 Adapter

```
//全局注册
LoadingHelper.getDefault().registerAdapter(VIEW_TYPE_LOADING, new LoadingAdapter());

//单个页面或控件注册
LoadingHelper loadingHelper = new LoadingHelper(this);
loadingHelper.registerAdapter(VIEW_TYPE_LOADING, new LoadingAdapter());
```

显示对应类型的视图

```
//显示指定类型的视图
loadingHelper.showView(viewType);

//对应类型 VIEW_TYPE_LOADING
loadingHelper.showLoadingView();

//对应类型 VIEW_TYPE_CONTENT
loadingHelper.showContentView();

//对应类型 VIEW_TYPE_ERROR
loadingHelper.showErrorView();

//对应类型 VIEW_TYPE_EMPTY
loadingHelper.showEmptyView();
```

如需重新加载数据

```
LoadingHelper.setRetryTask(new Runnable() {
  @Override
  public void run() {
    // 重新获取网路数据
  }
});

holder.retryTask.run();
```

### 拓展用法

#### 动态添加标题栏

创建标题栏的 Adapter，建议传入数据进行配置

```
public class TitleConfig {
  private String mTitleText;
  private Type mType;
  //省略 get set 方法
  public enum Type {
    BACK, NO_BACK
  }
}

public class TitleAdapter extends LoadingHelper.Adapter<TitleViewHolder> {
  private TitleConfig mConfig;

  public void setConfig(TitleConfig config) {
    mConfig = config;
  }

  public TitleConfig getConfig() {
    return mConfig;
  }

  @Override
  public TitleViewHolder onCreateViewHolder(@NonNull LayoutInflater inflater, @NonNull ViewGroup parent) {
    return new TitleViewHolder(new Toolbar(parent.getContext()));
  }

  @Override
  public void onBindViewHolder(@NonNull final TitleViewHolder holder) {
    if (mConfig != null) {
      // 根据配置更改视图
    }
  }
}
```

注册 Adapter，调用 addTitleView() 或 addHeaderView(int viewType) 增加标题栏

```
LoadingHelper.getDefault().registerAdapter(VIEW_TYPE_TITLE,new TitleAdapter());

final TitleAdapter adapter = loadingHelper.getAdapter(VIEW_TYPE_TITLE);
final TitleConfig config = new TitleConfig();
config.setTitleText("标题");
config.setType(TitleConfig.Type.BACK);
adapter.setConfig(config);
loadingHelper.addTitleView();
```

#### 动态更新数据

调用 notifyDataSetChanged()，会执行 Adapter.onBindViewHolder(...)

```
TitleAdapter adapter = loadingHelper.getAdapter(VIEW_TYPE_TITLE);
adapter.getConfig().setTitleText("正在加载中...");
adapter.notifyDataSetChanged();
```

#### 动态增加功能方法（拓展功能，使其不局限于显示视图）

实现 LoadingHelper.Method<T,VH extends ViewHolder> 接口，无返回值泛型传 Void，进行注册

```
// 无返回值
public class ShowTitleLoadingMethod implements LoadingHelper.Method<Void,TitleAdapter.TitleViewHolder> {
  @Override
  public Void execute(TitleAdapter.TitleViewHolder holder, Object... params) {
    boolean show = (boolean) params[0]; // 获取参数
    // 执行逻辑方法
    return null;
  }
}

// 有返回值
public class IsTitleLoadingMethod implements LoadingHelper.Method<Boolean,TitleAdapter.TitleViewHolder> {
  @Override
  public Boolean execute(TitleAdapter.TitleViewHolder holder, Object... params) {
    // 返回所需数值
  }
}

LoadingHelper.getDefault()
  .registerMethod(LoadingHelper.VIEW_TYPE_TITLE,"showTitleLoading",new ShowTitleLoadingMethod());
  .registerMethod(LoadingHelper.VIEW_TYPE_TITLE,"isTitleLoading",new IsTitleLoadingMethod());
```

用 LoadingHelper 或 ViewHolder 对象执行方法

```
//执行无返回值方法
loadingHelper.executeMethod(VIEW_TYPE_LOADING,"showTitleLoading",true);

//执行有返回值方法
boolean titleLoading = holder.getMethodReturnValue("isTitleLoading");
```

### 其它使用技巧

这里只是抛砖引玉，在 Activity 使用时，ContentAdapter 会得到 Activity.setContentView() 的视图，这里可以像基类一样进行一些默认的配置。 基类上的方法也可以通过本库来管理，进行二次封装，如果想更改方法内的逻辑，可以不动基类的代码，注册个新的方法类替换原有的逻辑即可，想新增全局的方法也可以不在基类添加代码。也可以用于标题栏的管理，想用哪个就注册哪个，也可以传入参数进行动态配置管理。灵活性很高，应用场景能有很多

## 感谢

- [luckbilly/Gloading](https://github.com/luckybilly/Gloading) 
- [drakeet/MultiType](https://github.com/drakeet/MultiType) 