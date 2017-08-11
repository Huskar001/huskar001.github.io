---
layout: post
title:  "抛弃findviewbyid--butterknife"
date:   2017-08-11 10:30:02 +0800
categories: android
tag: android
---
* content
{:toc}

以前写wpf时候，xml中命名的控件，在c#代码中都是可以直接引用的，觉得很方便，android中一直要使用findviewByid 一直觉得反人类。
最近发现一个ButterKnife的框架，可以不用写这些繁琐的findviewbyid了。

<img src="{{ '/styles/images/butterknife.png' | prepend: site.baseurl }}" width="293" />

老规矩 github地址 [https://github.com/JakeWharton/butterknife](https://github.com/JakeWharton/butterknife)

添加支持
----------

	apply plugin: 'com.jakewharton.butterknife'
	
	compile 'com.jakewharton:butterknife:8.8.0'
    annotationProcessor 'com.jakewharton:butterknife-compiler:8.8.0'
	
	classpath 'com.jakewharton:butterknife-gradle-plugin:8.8.0'
	
View绑定和资源绑定
-----------

    @BindView(R.id.title)//绑定view
    TextView title;
    @BindView(R.id.button)
    Button button;
    @BindString(R.string.name_str)//绑定字符串资源
    String name;
    @BindDrawable(R.drawable.favicon)//绑定图片资源
    Drawable graphic;
    @BindColor(R.color.colorPrimary)//绑定颜色资源
    int red;
    @BindDimen(R.dimen.btn_width)//绑定分辨率资源
    Float width;
	
别忘了在Activity中添加 

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        ButterKnife.bind(this);//
    }
	
其实内部生成如下代码

	public void bind(ExampleActivity activity) {
		activity.subtitle = (android.widget.TextView) activity.findViewById(2130968578);
		activity.footer = (android.widget.TextView) activity.findViewById(2130968579);
		activity.title = (android.widget.TextView) activity.findViewById(2130968577);
	}
	
非Activity绑定（NON-ACTIVITY BINDING）	
--------------------------

跟java代码一样,获取内部的view等资源，需要先获取view对象。
	public class FancyFragment extends Fragment {
		@BindView(R.id.button1) Button button1;
		@BindView(R.id.button2) Button button2;

		@Override 
		public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
			View view = inflater.inflate(R.layout.fancy_fragment, container, false);
			ButterKnife.bind(this, view);
			// TODO Use fields...
			return view;
		}
	}

在listview中使用的场景

	public class MyAdapter extends BaseAdapter {
		@Override 
		public View getView(int position, View view, ViewGroup parent) {
			ViewHolder holder;
			if (view != null) {
				holder = (ViewHolder) view.getTag();
			} else {
				view = inflater.inflate(R.layout.whatever, parent, false);
				holder = new ViewHolder(view);
				view.setTag(holder);
			}

			holder.name.setText("John Doe");
			// etc...
			return view;
		}

		static class ViewHolder {
			@BindView(R.id.title)
			TextView name;
			@BindView(R.id.job_title) 
			TextView jobTitle;

			public ViewHolder(View view) {
				ButterKnife.bind(this, view);
			}
		}
	}

ButterKnife.bind的调用可以被放在任何你想调用findViewById的地方。

提供的其他绑定API:

使用Activity作为跟布局在任意对象中进行绑定。如果你使用了类似MVC的编程模式，你可以对controller使用它的Activity用ButterKnife.bind(this, activity)进行绑定。

使用ButterKnife.bind(this)绑定一个布局的子布局。如果你在布局中使用了<merge>标签并且在自定义的控件构造时inflate这个布局，你可以在inflate之后立即调用它。或者，你可以在onFinishInflate()回调中使用它。

View List

你可以一次性将多个views绑定到一个List或数组中：

	@BindViews({ R.id.first_name, R.id.middle_name, R.id.last_name })
	List<EditText> nameViews;

apply函数，该函数一次性在列表中的所有View上执行一个动作：

	ButterKnife.apply(nameViews, DISABLE);
	ButterKnife.apply(nameViews, ENABLED, false);
	static final ButterKnife.Action<View> DISABLE = new ButterKnife.Action<View>() {
		@Override public void apply(View view, int index) {
			view.setEnabled(false);
		}
	};
	static final ButterKnife.Setter<View, Boolean> ENABLED = new ButterKnife.Setter<View, Boolean>() {
		@Override public void set(View view, Boolean value, int index) {
			view.setEnabled(value);
		}
	};

Android中的Property属性也可以使用apply方法进行设置：

	ButterKnife.apply(nameViews, View.ALPHA, 0.0f);

LISTENER 绑定
------------------

使用本框架，监听器能够自动的绑定到特定的执行方法上：

	@OnClick(R.id.submit)
	public void submit(View view) {
	  // TODO submit data to server...
	}
	
而监听器方法的参数都时可选的：

	@OnClick(R.id.submit)
	public void submit() {
		// TODO submit data to server...
	}
	
	指定一个特定的类型，Butter Knife也能识别：
	
	@OnClick(R.id.submit)
	public void sayHi(Button button) {
		button.setText("Hello!");
	}
	
	可以指定多个View ID到一个方法上，这样，这个方法就成为了这些View的共同事件处理。
	
	@OnClick({ R.id.door1, R.id.door2, R.id.door3 })
	public void pickDoor(DoorView door) {
		if (door.hasPrizeBehind()) {
			Toast.makeText(this, "You win!", LENGTH_SHORT).show();
		} else {
			Toast.makeText(this, "Try again", LENGTH_SHORT).show();
		}
	}
	
自定义View时，绑定事件监听不需要指定ID 因为本来就是button

	public class FancyButton extends Button {
		@OnClick
		public void onClick() {
			// TODO do something!
		}
	}

绑定重置
------------------

Fragment的生命周期与Activity不同。在Fragment中，如果你在onCreateView中使用绑定，那么你需要在onDestroyView中设置所有view为null。为此，ButterKnife返回一个Unbinder实例以便于你进行这项处理。在合适的生命周期回调中调用unbind函数就可完成重置。

	public class FancyFragment extends Fragment {
		@BindView(R.id.button1) Button button1;
		@BindView(R.id.button2) Button button2;
		private Unbinder unbinder;

		@Override public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
			View view = inflater.inflate(R.layout.fancy_fragment, container, false);
			unbinder = ButterKnife.bind(this, view);
			// TODO Use fields...
			return view;
		}

		@Override public void onDestroyView() {
        super.onDestroyView();
        unbinder.unbind();
		}
	}

可选绑定
--------------------

在默认情况下， @bind和监听器的绑定都是必须的，如果目标view没有找到的话，Butter Knife将会抛出个异常。

如果你并不想使用这样的默认行为而是想创建一个可选的绑定，那么你只需要在变量上使用@Nullable注解或在函数上使用@Option注解。

注意：任何名为@Nullable的注解都可以使用在变量上。但还时强烈建议使用Android注解库中的@Nullable。使用这个库对你的代码有很多好处。
关于该库的详情，可以点击此处：[Android Tools Project](http://tools.android.com/tech-docs/support-annotations)

	@Nullable @BindView(R.id.might_not_be_there) 
	TextView mightNotBeThere;

	@Optional @OnClick(R.id.maybe_missing) 
	void onMaybeMissingClicked() {
		// TODO ...
	}

对于包含多个方法的监听
----------------------

当一个监听器包含多个回调函数时，使用函数的注解能够对其中任何一个函数进行绑定。每一个注解都会绑定到一个默认的回调。你也可以使用callback参数来指定一个其他函数作为回调。

	@OnItemSelected(R.id.list_view)
	void onItemSelected(int position) {
		// TODO ...
	}

	@OnItemSelected(value = R.id.maybe_missing, callback = NOTHING_SELECTED)
	void onNothingSelected() {
		// TODO ...
	}

BONUS
---------------

如果还是想用findviewbyid 那么butterknife提供了代替的方法，少了强制转换，强迫症的福音。而且是静态方法。

	View view = LayoutInflater.from(context).inflate(R.layout.thing, null);
	TextView firstName = ButterKnife.findById(view, R.id.first_name);
	TextView lastName = ButterKnife.findById(view, R.id.last_name);
	ImageView photo = ButterKnife.findById(view, R.id.photo);
