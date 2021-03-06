# TestDOTDemo
a DoT widget demo

#Android 页面选择小圆点（页面显示自定义控件）

###作用：自定义小控件


###效果图：
![](http://i.imgur.com/8inLFo4.png)

###style文件
    <?xml version="1.0" encoding="utf-8"?>
    <resources>
    <declare-styleable name="AbSlidingButtonStyle">
        <attr name="btn_bottom" format="integer|reference" />
        <attr name="btn_frame" format="integer|reference" />
        <attr name="btn_mask" format="integer|reference" />
        <attr name="btn_unpressed" format="integer|reference" />
        <attr name="btn_pressed" format="integer|reference" />
    </declare-styleable>

    <declare-styleable name="BaseDoT">
        <attr name="circleColor" format="color|reference" />
    </declare-styleable>

    <declare-styleable name="BaseDoTView">
        <attr name="slideCount" format="integer|reference" />
        <attr name="selectPosition" format="integer|reference" />
        <attr name="selectedIndicatorColor" format="color|reference" />
        <attr name="unselectedIndicatorColor" format="color|reference" />
        <attr name="circleRadius" format="dimension|reference" />
    </declare-styleable>

    <declare-styleable name="BaseTopBackCenterHeader">
        <attr name="headerTitle" format="reference|string" />
        <attr name="headerTitleColor" format="color|reference" />
        <attr name="headerShowLeft" format="boolean" />
        <attr name="headerLeftTitle" format="reference|string" />
    </declare-styleable>
    </resources>

###Dot（自定义小圆点）文件
    public class Dot extends ImageView {
    /**
     * 放大放大倍数
     */
    public static final float SCALE = 1.0f;
    /**
     * 透明度
     */
    public static final int ALPHA = 255;

    private float[] scaleFloats = new float[]{SCALE, SCALE, SCALE};

    private int[] alphas = new int[]{ALPHA, ALPHA, ALPHA,};

    public Dot(Context context) {
        this(context, null);
    }

    public Dot(Context context, AttributeSet attrs) {
        super(context, attrs);
        init(context, attrs);
    }

    /**
     * 小圆初始化设置
     *
     * @param context
     * @param attrs
     */
    private void init(Context context, AttributeSet attrs) {
        paint = new Paint();
        paint.setAntiAlias(true);// 设置画笔的锯齿效果。 true是去除，大家一看效果就明白了
        setLayoutParams(new LinearLayout.LayoutParams(LinearLayout.LayoutParams.WRAP_CONTENT, LinearLayout.LayoutParams.WRAP_CONTENT));
        if (null != attrs) {
            TypedArray arr = context.obtainStyledAttributes(attrs, R.styleable.BaseDoT);
            try {
                int selectedIndicatorColor = arr.getColor(R.styleable.BaseDoT_circleColor, getResources().getColor(R.color.cl_Grid));
                this.setColor(selectedIndicatorColor);
            } finally {
                arr.recycle();
            }
        }
    }

    /**
     * Dot默认宽度或高度
     */
    private float defaultSize = 15;

    /**
     * 修改DoT默认尺寸
     *
     * @param circleRadius
     */
    public void setCircleRadius(float circleRadius) {
        if (circleRadius > 0)
            defaultSize = circleRadius * 3;
    }

    /**
     * 小圆画笔
     */
    private Paint paint = null;

    public void setColor(int color) {
        if (paint != null)
            paint.setColor(color);
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        float radius = getWidth() / 3;
        float x = getWidth() / 2;
        float y = getHeight() / 2;
        canvas.save();//保存设置
        float translateX = x;
        canvas.translate(translateX, y);//平移画笔起始点
        canvas.scale(scaleFloats[0], scaleFloats[0]);//放大
        paint.setAlpha(alphas[0]);//设置透明度
        canvas.drawCircle(0, 0, radius, paint);//画圆
        canvas.restore();
    }


    /**
     * 当控件的父元素正要放置该控件时调用
     * 根据父容器传递跟子容器的大小要求来确定子容器的大小
     *
     * @param widthMeasureSpec
     * @param heightMeasureSpec
     */
    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        int width = measureDimension(ABTextUtil.dip2px(getContext(), defaultSize), widthMeasureSpec);
        int height = measureDimension(ABTextUtil.dip2px(getContext(), defaultSize), heightMeasureSpec);
        setMeasuredDimension(width, height);
    }

    /**
     * 根据父容器传递跟子容器的大小要求来确定子容器的大小
     *
     * @param defaultSize
     * @param measureSpec
     * @return
     */
    private int measureDimension(int defaultSize, int measureSpec) {
        int result = defaultSize;
        int specMode = MeasureSpec.getMode(measureSpec);
        int specSize = MeasureSpec.getSize(measureSpec);
        if (specMode == MeasureSpec.EXACTLY) {
            result = specSize;
        } else if (specMode == MeasureSpec.AT_MOST) {
            result = Math.min(defaultSize, specSize);
        } else {
            result = defaultSize;
        }
        return result;
     }
    }

###DoTView小圆点容器（自定义控件：使用时直接在布局文件中调用）
    public class DoTView extends BaseDoTView {
    /**
     * 小圆点集合
     */
    protected List<Dot> mDots;

    public DoTView(Context context) {
        super(context);
    }

    public DoTView(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    @Override
    protected void initViews() {
        LayoutInflater.from(getContext()).inflate(R.layout.default_indicator, this);
    }

    @Override
    public void initialize(int slideCount, int first_page_num) {
        mDots = new ArrayList<>();
        mSlideCount = slideCount;

        for (int i = 0; i < slideCount; i++) {
            Dot dot = new Dot(getContext());
            dot.setColor(getUnselectedIndicatorColor());
            LayoutParams params = new LayoutParams(LayoutParams.WRAP_CONTENT, LayoutParams.WRAP_CONTENT);
            dot.setCircleRadius(getCircleRadius());
            addView(dot, params);
            mDots.add(dot);
        }
        setSelectPosition(first_page_num);
    }

    @Override
    public void setSelectPosition(int index) {
        ABLogUtil.i("index------------------" + index);
        setCurrentposition(index);
        for (int i = 0; i < mSlideCount; i++) {
            if (i == index)
                mDots.get(i).setColor(getSelectedIndicatorColor());
            else
                mDots.get(i).setColor(getUnselectedIndicatorColor());
            mDots.get(i).postInvalidate();
        }
        postInvalidate();//刷新界面
    }
    }

###default_indicator.xml（DoTView控件的布局文件）
    <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="wrap_content"
    android:layout_height="fill_parent"
    android:gravity="center"
    android:orientation="horizontal"
    android:layout_gravity="center"/>
###BaseDoTView（Base文件：初始化设置文件）
    public abstract class BaseDoTView extends LinearLayout {

    /**
     * 小圆点个数
     */
    protected int mSlideCount;
    /**
     * 默认颜色
     */
    private final static int DEFAULT_COLOR = 1;
    /**
     * 被选中颜色
     */
    private int selectedDotColor = DEFAULT_COLOR;
    /**
     * 未被选中颜色
     */
    private int unselectedDotColor = DEFAULT_COLOR;
    /**
     * 当前选中位置
     */
    private int mCurrentposition;
    /**
     * 小圆半径
     */
    private int circleRadius;

    public BaseDoTView(Context context) {
        this(context, (AttributeSet) null);
    }

    public BaseDoTView(Context context, AttributeSet attrs) {
        super(context, attrs);
        this.initViews();
        setGravity(Gravity.CENTER);
        if (null != attrs) {
            TypedArray arr = context.obtainStyledAttributes(attrs, R.styleable.BaseDoTView);
            try {
                //被选中小点的颜色
                int selectedIndicatorColor = arr.getColor(R.styleable.BaseDoTView_selectedIndicatorColor, getResources().getColor(R.color.cl_Grid));
                this.setSelectedIndicatorColor(selectedIndicatorColor);
                //未被选中的小点颜色
                int UnselectedIndicatorColor = arr.getColor(R.styleable.BaseDoTView_unselectedIndicatorColor, getResources().getColor(R.color.cl_gray));
                this.setUnselectedIndicatorColor(UnselectedIndicatorColor);
                //小点半径
                int circleRadius = arr.getInt(R.styleable.BaseDoTView_circleRadius, 0);
                if (circleRadius > 0)
                    setCircleRadius(circleRadius);
                //小点数
                int slideCount = arr.getInt(R.styleable.BaseDoTView_slideCount, 0);
                //小点位置
                int selected = arr.getInt(R.styleable.BaseDoTView_selectPosition, 0);
                if (slideCount > 0) {
                    this.initialize(slideCount, selected);
                }
            } finally {
                arr.recycle();
            }
        }
    }

    /**
     * 设置小点个数及默认选中
     *
     * @param slideCount     小点个数
     * @param first_page_num 默认选中
     */
    public abstract void initialize(int slideCount, int first_page_num);

    /**
     * 设置默认选中
     *
     * @param index The index of the page that became selected
     */
    public abstract void setSelectPosition(int index);

    /**
     * 初始化视图
     */
    protected abstract void initViews();

    /**
     * 设置选中小点颜色
     *
     * @param selectedDotColor
     */
    public void setSelectedIndicatorColor(int selectedDotColor) {
        this.selectedDotColor = selectedDotColor;
    }

    /**
     * 获取选中小点颜色
     *
     * @return
     */
    public int getSelectedIndicatorColor() {
        return selectedDotColor;
    }

    /**
     * 设置未被选中小点的颜色
     *
     * @param unselectedDotColor
     */
    public void setUnselectedIndicatorColor(int unselectedDotColor) {
        this.unselectedDotColor = unselectedDotColor;
    }

    /**
     * 获取未被选中小点的颜色
     *
     * @return
     */
    public int getUnselectedIndicatorColor() {
        return unselectedDotColor;
    }

    /**
     * 获取当前选中小点位置
     *
     * @return
     */
    public int getCurrentposition() {
        return mCurrentposition;
    }

    /**
     * 设置选中小点
     *
     * @param mCurrentposition
     */
    public void setCurrentposition(int mCurrentposition) {
        this.mCurrentposition = mCurrentposition;
    }

    /**
     * 设小点半径
     *
     * @param circleRadius
     */
    public void setCircleRadius(int circleRadius) {
        this.circleRadius = circleRadius;
    }

    /**
     * 获取小点半径
     *
     * @return
     */
    public int getCircleRadius() {
        return circleRadius;
    }
    }

###MainActivity（主页面显示控件效果）
    public class MainActivity extends AppCompatActivity implements View.OnClickListener {

    private DoTView doTView;
    private Dot dot;
    private int index = 0;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        doTView = (DoTView) findViewById(R.id.dotView);
        dot = (Dot) findViewById(R.id.dot);
        index = doTView.getCurrentposition();
    }

    @Override
    public void onClick(View v) {
        switch (v.getId()) {
            case R.id.btn:
                if (ABNumberUtil.isOdd(index))
                    dot.setColor(getResources().getColor(R.color.cl_Blue));
                else dot.setColor(getResources().getColor(R.color.cl_Red));
                dot.postInvalidate();
                if (index < doTView.getChildCount() - 1)
                    index++;
                else if (index == doTView.getChildCount() - 1)
                    index = 0;
                doTView.setSelectPosition(index);
                break;
            default:
                break;
        }
    }
    }

###activity_main.xml（MainActivity页面的布局文件）
    <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:gravity="center"
    android:orientation="vertical"
    tools:context="com.admin.MainActivity">

    <com.admin.dot.Dot
        android:id="@+id/dot"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        app:circleColor="@color/cl_Blue" />

    <com.admin.dot.DoTView
        android:id="@+id/dotView"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="@color/cl_gray"
        android:orientation="horizontal"
        android:paddingBottom="5dp"
        android:paddingTop="5dp"
        app:selectPosition="2"
        app:selectedIndicatorColor="#068BF2"
        app:slideCount="5"
        app:unselectedIndicatorColor="#FF0000" />

    <Button
        android:id="@+id/btn"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:onClick="onClick"
        android:text="btn" />

    </LinearLayout>
