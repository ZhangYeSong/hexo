---
title: 深夜爬坑——Kotlin+DataBinding方法命名引起绑定问题
date: 2018-11-23 01:50:02
categories:
 - Android
tags:
 - Kotlin
---
已经夜里一点多了，还是想分享下。
```
class WalletViewModel : ViewModel() {
    var balance: MutableLiveData<String> = MutableLiveData()

    lateinit var disposable: Disposable

    init {
        balance.value = "未知"
    }

    fun getBalance() {
        disposable = RetrofitFactory.retrofitApi.getRestMoney(OTL.getToken().bearerAccessToken)
                .observeOn(AndroidSchedulers.mainThread())
                .subscribeOn(Schedulers.io())
                .subscribe({
                    balance.value = it.balance
                }, {
                    showToast("获取余额失败")
                    balance.value = "未知"
                })
    }

    override fun onCleared() {
        disposable.dispose()
    }
}
```
这样一段代码死活绑定报错，java.lang.RuntimeException: Found data binding errors.
****/ data binding error ****msg:Could not find accessor .....
搜了下stackoverflow，说是绑定的对象一定要有get方法，可是我用的kotlin，非private变量默认就有get、set方法。这下一下子失了智，先是怀疑LiveData的问题，可是Google明确说了现在AS3.1以后可以用LiveData替代ObserveFiled。
完了，继续死磕，但是毫无头绪。
试着强行加get方法，当然还是报错，但是突然发现我加的get方法和我下面获取数据的getBalance()方法重名！一下子茅塞顿开，DataBinding默认去拿这个方法来获取我的liveData当然获取不到了。
最近写Kotlin+DataBinding+LiveData+MVVM有点上头，熬了几天终于有了点渐入佳境的赶脚。不说了，快两点了还没洗澡
