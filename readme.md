# Create UI component by nib - 來用nib拉UI！
對iOS工程師來說，撰寫view已經是跟陽光空氣水一樣重要的事情了，目前iOS開發環境所支持的view撰寫方式，不外乎兩種：

* 純code
* 利用nib或storyboard等Interface Builder(IB)拖拉

雖然使用IB有很多壞處，像是無法code review、merge容易有問題、還有無法完成很多view的效果，像是動畫等等。但是因為它所見即所得，還有容易拉Auto Layout等等特性，還是讓IB有它不可或缺的重要性。撇開storyboard不談，如果我們想要客制化一個view，叫SocialToolsView，想使用nib來拉Auto Layout，我們會需要這樣：

```swift
class SocialToolsView: UIView {
   required init(coder aDecoder: NSCoder) {
      super.init(coder: aDecoder)
      NSBundle.mainBundle().loadNibNamed("SocialToolsView", owner: self, options: nil)
      self.addSubview(self.view); 
   }
}
```

在subclass裡面覆寫init。

或是：

```swift
let view = NSBundle.mainBundle().loadNibNamed("SocialToolsView", owner: self, options: nil)?[0] as? SocialToolsView
```

在需要這個view的時候在把nib抓出來。

不管用那一種方法，只要view一多，就需要寫很多重覆的code。

## No more redundancy 
我們希望可以每一次取用SocialToolsView時，都不用寫上面那一堆冗長的code，也不想要每次客制化view的時候，都要一直覆寫init。在這邊，我們可以利用extension跟generic的技巧，來避免這些重覆的code，如下：

```swift
protocol NibInstantiable {}

extension UIView: NibInstantiable {}

extension NibInstantiable where Self: UIView {
    static func instantiateFromNib() -> Self {
        if let view = Bundle(for: self).loadNibNamed(String(describing: self), owner: nil, options: nil)?[0] as? Self {
            return view
        } else {
            assert(false, "The nib named \(self) is not found")
            return Self()
        }
    }
}
```

這是一個static的function，功用就是從bundle裡面讀取跟class名稱一樣的nib，並且回傳實例化view的物件。利用這個extension，我們就可以做到：

```swift
let toolView: SocialToolsView = SocialToolsView.instantiateFromNib()
```

如果我們新增一個CustomizedView，並且加入名稱一樣的nib，我們就可以依樣畫葫蘆：

```swift
let view: CustomizedView = CustomizedView.instantiateFromNib()
```

不用再寫冗長又看不太懂的code了！🍺

## Dive deeper 
仔細研究一下這份code，可以發現這個generic type T，其實是透過回傳值去infer的，這也是為甚麼我們在assign這個view的時候(let toolView: SocialToolsView = ...)，一定要標註view的type，目地就是讓compiler能夠正確infer這個T。

*update:* 

感謝[Hsu Li-Heng](https://www.facebook.com/yesleon?fref=gc&dti=903202893043760)大大提供的建議，利用protocol的extension能夠infer自己的特點，加上有條件的protocol extension，可以解決上述需要在code裡面explicit指定類別的問題。

Ref: [Using Self in Swift Class Extensions](https://medium.com/@victor.pavlychko/using-self-in-swift-class-extensions-6421dab02587)

另外我們也針對這個generic做限制(T: UIView)，目的是為了在nib找不到的時候，可以fallback到回傳T()。也因為T已經被限制是一種UIView，未來在這個extension裡面如果要針對T物件做UI相關的處理，也會變得非常容易。🍺🍺

## Appendix: Setup your nib
以下是這個repository的nib設定：

![](readme/Screen%20Shot%202018-06-01%20at%2000.12.28.png)

_ps. 這是一個social tool view，讓你擺在任何想要social的元件上。_


記得把view的class設定成你的class：  

![](readme/Screen%20Shot%202018-06-01%20at%2000.12.42.png)


我們的class跟nib總是成對出現，在trace code時可以先看圖(IB)再看code：  

![](readme/Screen%20Shot%202018-06-01%20at%2000.45.36.png)



