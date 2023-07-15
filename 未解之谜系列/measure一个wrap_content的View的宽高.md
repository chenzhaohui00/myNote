## 如何主动measure一个wrap_content的View的宽/高

inflate一个view出来，这个view的宽高是wrap_content的，measure以后发现getMeasuredWidth/getMeasuredHeight的值不对。

场景如下：

1. 我们要在一个控件的上面show一个popupWindow，因此需要在show之前计算popup的宽高。

2. 经过搜索得到的一个比较合理的计算方式如下：

   ```java
   public void showAboveOf(View view, int xOff, int yOff) {
       mContentView.measure(makeDropDownMeasureSpec(getWidth()), makeDropDownMeasureSpec(getHeight()));
       int popupWidth = mContentView.getMeasuredWidth();
       int popupHeight = mContentView.getMeasuredHeight();
       int[] location = new int[2];
       view.getLocationOnScreen(location);
       showAtLocation(view, Gravity.NO_GRAVITY, location[0] + view.getWidth() / 2 - popupWidth / 2 + xOff,
               location[1] - popupHeight + yOff);
   }
   
   public static int makeDropDownMeasureSpec(int measureSpec) {
       int mode;
       if (measureSpec == ViewGroup.LayoutParams.WRAP_CONTENT) {
           mode = View.MeasureSpec.UNSPECIFIED;
       } else {
           mode = View.MeasureSpec.EXACTLY;
       }
       return View.MeasureSpec.makeMeasureSpec(View.MeasureSpec.getSize(measureSpec), mode);
   }
   ```

3. 我们要show的popup里面有一个ListView，高度是wrap_content的，所以这个contentView也是wrap_content的，整个popup也是wrap_content的。这里就发现用这个makeDropDownMeasureSpec测量出来的高度不对，一个明显应该是300多的高度，measure出来是100多。

4. 有朋友提醒我，WRAP_CONTENT的时候应该用MeasureSpec.AT_MOST，但是显然不是这个问题，因为UNSPECIFIED是不限制，AT_MOST是不能超过parent，现在是测量出来的高度小了，改成AT_MOST显然也没用，试了一下果然没用。

5. 最后赶工也没解决，就手动根据data有几条，算一下高度，弄的差不多了。

