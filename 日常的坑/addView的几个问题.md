## addView不显示

我在一个自定义的组合控件里面addView，一次性add多个view。发现必须要用handler.post才能add上去，否则add完也不显示。

就是下面这一段代码，往一个RadioGroup里面根据配置添加n个RadioButton

```kotlin
buttons!!.forEachIndexed { index, btnText ->
    val rb = RadioButton(context)
    rb.buttonDrawable = null
    rb.setBackgroundResource(R.drawable.selector_subject_radio_button_bg)
    rb.text = btnText
    if (btnBold) {
        //设置为粗体
        rb.typeface =
            Typeface.createFromAsset(context.assets, "fonts/SourceHanSansCN-Medium.otf")
    }
    val lp = RadioGroup.LayoutParams(btnWidth, LayoutParams.MATCH_PARENT)
    rb.gravity = Gravity.CENTER
		//lp.gravity = Gravity.CENTER
    if (index > 0) {
        lp.marginStart = btnMargin
    }
    rb.tag = buttonTags!![index]
    rb.setOnCheckedChangeListener { radioBtn, isChecked ->
        if (isChecked) {
            buttonsListener?.invoke(radioBtn.tag.toString())
        }
    }
    if (pendingCheckRbIndex == index || pendingCheckRbTag == rb.tag) {
        rb.isChecked = true
    }
    handler.post {
        rg.addView(rb, lp)
    }
    Timber.d("add RadioButton, text:$btnText, tag:${rb.tag}")
}
```



## add RadioButton 无法居中

RadioButton中有文字，发现文字无法居中。在xml中写`android:gravity="Center"`就可以居中了，但是 new 的`RadioButton`去 addView，在LayoutParams中设置却无效。最后发现radioButton最终继承的TextView自己是有一个setGravity，调用此方法设置即可。就是上面代码中的这两行：

```kotlin
rb.gravity = Gravity.CENTER
//lp.gravity = Gravity.CENTER
```

要使用上面的 radioButton 的这个方法，而不是设置到 LayoutParams 中。



## RadioButton必须都有ID

一般在手动往RadioGroup中添加RadioButton时，可能不会给每个radioButton添加Id，然后就会发现在某些情况下，会发生一个RadioGroup中的多个RadioButton可以同时被选中。比如我这次在一个自定义的组合控件中，根据attributeSet或调用的方法传入的一些参数动态添加RadioButton，然后在某个页面中的某个这个控件，他的子组件的某一个无法被uncheck，即使check了其他rb，这个rb也还是选中状态。