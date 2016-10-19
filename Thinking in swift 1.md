# Thinking in Swift, Part 1: 조랑말 구하기

난 종종 ObjC 코드를 Swift로 변환하는 데 애쓰고 있는 Swift 입문자들을 만난다. 하지만 Swift 코딩을 시작하는 데 있어서 가장 어려운 점은 문법이 아니라 ObjC에는 쓰지 않던 새로운 스위프트의 개념들을 쓰기 위해 사고의 방식을 바꾸는 일이다. 

이번 시리즈에서 우리는 ObjC 예제 코드를 가져와 개념들을 하나씩 소개하면서 Swift로 바꿔보려 한다.

> *이번 파트에서 얘기할 주제 : 옵셔널, 강제 언랩핑 옵셔널, 조랑말, `if let`, `guard`, 그리고 :cake:*

*이 포스트는 시리즈의 일부입니다. 나머지 부분은 여기서 읽으실 수 있습니다 (원문)  : [파트 1](http://alisoftware.github.io/swift/2015/09/06/thinking-in-swift-1/), [파트 1 부록](http://alisoftware.github.io/swift/2015/09/14/thinking-in-swift-1-addendum/), [파트 2](http://alisoftware.github.io/swift/2015/09/20/thinking-in-swift-2/), [파트 3](http://alisoftware.github.io/swift/2015/10/03/thinking-in-swift-3/), [파트  4](http://alisoftware.github.io/swift/2015/10/11/thinking-in-swift-4/)* 

### ObjC 코드

자, 당신이 어떤 아이템들의 리스트 —각각의 아이콘, 제목, URL을 가지고 있는— 를 만들고 싶다고 생각해보자. (예를 들어 차후에 TableView에 보여줄만한) ObjC 코드는 이런 형태일 것이다.

```objective-c
@interface ListItem : NSObject
@property(strong) UIImage* icon;
@property(strong) NSString* title;
@property(strong) NSURL* url;
@end

@implementation ListItem
+(NSArray*)listItemsFromJSONData:(NSData*)jsonData {
    NSArray* itemsDescriptors = [NSJSONSerialization JSONObjectWithData:jsonData options:0 error:nil];
    
    NSMutableArray* items = [NSMutableArray new];
    for (NSDictionary* itemDesc in itemsDescriptors) {
        ListItem* item = [ListItem new];
        item.icon = [UIImage imageNamed:itemDesc[@"icon"]];
        item.title = itemDesc[@"title"];
        item.url = [NSURL URLWithString:itemDesc[@"url"]];
        [items addObject:item];
    }
    return [items copy];
}
@end
```

좋다, 꽤 표준적이라 할만한 코드다.

### Swift로 직역해보기

이제 어떤 Swift 입문자가 이 코드를 기본적인 수준에서 변환한다고 생각해보자 :

```swift
class ListItem {
    var icon: UIImage?
    var title: String = ""
    var url: NSURL!
    
    static func listItemsFromJSONData(jsonData: NSData?) -> NSArray {
        let jsonItems: NSArray = try! NSJSONSerialization.JSONObjectWithData(jsonData!, options: []) as! NSArray
        let items: NSMutableArray = NSMutableArray()
        for itemDesc in jsonItems {
            let item: ListItem = ListItem()
            item.icon = UIImage(named: itemDesc["icon"] as! String)
            item.title = itemDesc["title"] as! String
            item.url = NSURL(string: itemDesc["url"] as! String)!
            items.addObject(item)
        }
        return items.copy() as! NSArray
    }
}
```

Swift에 조금은 경험이 있는 사람은 이미 여러 악취 코드(code smell)들을 발견했을 것이고, 아마 이 코드를 읽은 Swift 경력자들은 모두 심장 마비로 이미 사망했을 거다.

### 무엇이 잘못 된 걸까?

위 코드에서 악취 코드로 보이는 첫 번째 부분은 Swift 입문자들이 종종 범하는 매우 안 좋은 버릇이다 : 어디에서나 강제 언랩핑 옵셔널 (*implicitly-unwrapped optionals*, `value!`), 강제 형 변환 (*force-cast*, `value as! String`) 그리고 강제 try (*force-try*, `try!`)를 사용하는 부분 말이다.

**옵셔널은 우리의 친구이다** : 값이 언제 `nil`을 갖는 지, 또 이런 상황을 어떻게 처리해야 하는지 생각하도록 만든다는 점에서 옵셔널은 매우 좋다. 예를 들어 "만약 아이콘이 없을 때는 무엇을 표시하지? TableViewCell에 placeholder를 사용해야 하나? 아니면 완전히 다른 템플릿의 셀을 사용해야 하나?"

우리가 ObjC를 쓸 때는 가끔 잊어먹고 고려하지 못한 케이스들이 있었지만, 스위프트는 그것들을 까먹지 않도록 도와준다. 따라서 `nil` 값이었던 부분을 **강제 언랩핑으로 날려버리고 당신의 코드를 크래쉬내어 이런 이점들을 날리는 건 부끄러운 일이다.**

> 당신이 무슨 일을 하고 있는지 정확히 알고 있을 때를 제외하고는, **절대로** 값을 강제 언랩핑하지 말아야 한다. 단순히 컴파일러를 만족하게 하기 위해 `!`을 붙일 때마다 조랑말 한 마리를 죽이고 있다는 사실을 기억해라 🐴.

애석하게도, Xcode는 이런 실수를 권장한다. 왜냐하면, 이런 에러가 났을 때: "*value of optional type ‘NSArray?’ not unwrapped. Did you mean to use ! or ??*", fix-it 제안으로 뜨는 것은… 끝에 `!`을 붙이는 것이다*🙀*. 오, Xcode여, 뉴비가 여기 있네!

### 조랑말들을 살려보자

이런 안 좋은  `!`들을 모두 없애려면 어떻게 해야 할까? 몇 가지 기술들을 소개한다 :

- 옵셔널 바인딩을 사용해라. `if let x = optional { /* use x */ }`
- 타입 캐스팅 실패 시  `nil`을 던지는  `as!` 대신, `as?`를 사용해라. 당연히 `if let` 과 혼합하여 쓸 수 있다.
- `try!`는  해당 부분에서 에러 발생 시 `nil`을 리턴한다. 이 대신 `try?`을 사용할 수 있다.


자, 이제 이런 규칙을 활용했을 때 코드가 어떻게 변하는지 보자.

```swift
class ListItem {
  var icon: UIImage?
  var title: String = ""
  var url: NSURL!

  static func listItemsFromJSONData(jsonData: NSData?) -> NSArray {
      if let nonNilJsonData = jsonData {
          if let jsonItems: NSArray = (try? NSJSONSerialization.JSONObjectWithData(nonNilJsonData, options: [])) as? NSArray {
              let items: NSMutableArray = NSMutableArray()
              for itemDesc in jsonItems {
                  let item: ListItem = ListItem()
                  if let icon = itemDesc["icon"] as? String {
                      item.icon = UIImage(named: icon)
                  }
                  if let title = itemDesc["title"] as? String {
                      item.title = title
                  }
                  if let urlString = itemDesc["url"] as? String {
                      if let url = NSURL(string: urlString) {
                          item.url = url
                      }
                  }
                  items.addObject(item)
              }
              return items.copy() as! NSArray
          }
      }
      return [] // In case something failed above
  }
}
```
### 운명의 피라미드

아쉬운 점은, 이렇게 `if let`을 많이 사용하다 보면 코드가 오른쪽으로 계속 밀리게 되고 결국엔 악명 높은 [운명의 피라미드(pyramid of doom)](https://en.wikipedia.org/wiki/Pyramid_of_Doom)를 불러오게 된다 *(두둥)*.

Swift에는 이런 현상을 줄일 수 있는 몇 가지 구조가 있다 :

- 여러 줄의 `if let` 한 줄로 합치기 : `if let x = opt1, y = opt2`
- `guard` 사용하기 : 조건이 충족되지 않으면 함수를 바로 빠져나올 수 있게 해준다. 따라서 함수의 나머지 부분이 오른쪽으로 밀리지 않는다.

이 코드 정리 방법을 통해 발견할 수 있는 여러 문제 요소들 —단순히 `let items = NSMutableArray()` 이런 식으로 쓴 부분들— 을 제거하고 `guard`문을 통해 우리가 사용한 JSON이 정말 `NSDictionary` 객체의 배열이 맞는지 확인할 수 있는 장점을 이용해보자. 마지막으로, ObjC의 `NSArray`보다는 더 'Swift-er'한 `[ListItem]`을 리턴타입으로 사용해보자 :

```swift
class ListItem {
  var icon: UIImage?
  var title: String = ""
  var url: NSURL!

  static func listItemsFromJSONData(jsonData: NSData?) -> [ListItem] {
      guard let nonNilJsonData = jsonData,
          let json = try? NSJSONSerialization.JSONObjectWithData(nonNilJsonData, options: []),
          let jsonItems = json as? Array<NSDictionary>
          else {
              // If we failed to unserialize the JSON
              // or that JSON wasn't an Array of NSDictionaries,
              // then bail early with an empty array
              return []
      }

      var items = [ListItem]()
      for itemDesc in jsonItems {
          let item = ListItem()
          if let icon = itemDesc["icon"] as? String {
              item.icon = UIImage(named: icon)
          }
          if let title = itemDesc["title"] as? String {
              item.title = title
          }
          if let urlString = itemDesc["url"] as? String, let url = NSURL(string: urlString) {
              item.url = url
          }
          items.append(item)
      }
      return items
  }
}
```
`guard`문은 input 값들이 유효한지 확인하는 부분을 함수의 맨 처음 부분으로 몰아주기 때문에 정말 좋다. 따라서 나머지 부분에서는 이 값들 확인 때문에 성가실 일이 없게 된다. 만약 input이 기대한 값이 아니라면 우리는 함수를 일찍 종료하도록 하면 되고, 이것은 우리가 예상한 부분, 올바른 부분만 신경 쓸 수 있도록 도와준다.

### Swift가 ObjC 보다 더 간결하다며?

![the cake is a lie](http://alisoftware.github.io/assets/the-cake-is-a-lie.png){: .text-center} 

음, ObjC 코드보다 더 복잡해 보이는 건 맞다. 하지만 앞으로 이어질 part 2에서 더 간결하게 만들 예정이니 걱정할 필요는 없다. 

하지만 가장 중요한 건, **이 코드가 ObjC의 코드보다 훨씬 더 안전하다는 것이다.** 사실 ObjC의 코드는 수많은 안전 테스트를 빼먹었기 때문에 짧았을 뿐이다. 비록 굉장히 평범해 보이지만 **이 ObjC 코드는 당장 여러 가지 상황에서 크래쉬를 낼 수 있다.** 예를 들어 유효하지 않은 JSON을 주거나, 그중 하나라도 String을 가지는 dictionary 배열이 아니었을 때 말이다. (만약 누군가 JSON을 만들 때 "icon" 키의 value 값으로 `String`이 아니라 아이콘을 가지고 있다/없다를 뜻하는 Boolean을 사용한다면..) **우리는 ObjC에서는 이런 케이스들을 망각하고 있었다.** 왜냐하면 Swift가 그런 케이스들을 고려하도록 도와주는 데 반해 ObjC는 그렇지 않기 때문이다.

따라서 물론 ObjC가 더 짧긴 하다 : 하지만 그 이유는 단순히 이런 일들의 처리를 까먹었기 때문이다! 만약 크래쉬가 나더라도 상관없다고 한다면, 코드 길이를 줄이는 건 쉬운 일이다. 도로를 막고 있는 장애물들이 없을 때 운전이 쉬워지는 건 당연하다. 하지만 그건 당신이 조랑말을 죽이는 방법이기도 하다.

### 결론

Swift는 안전하게 설계되었다. 옵셔널을 무시하고 모두 강제 언랩핑해서는 안된다. 당신이 Swift 코드에서 `!`을 본다면, 이건 대부분 악취 코드일 것이고 뭔가 잘못하고 있다는 걸 항상 생각해야 한다.

다가올 part 2에서는 이 Swift 코드를 더 가볍게 만드는 방법을 다루고 `for`반복문과 `if-let`  대신 `map`과 `flat-map` 으로 바꿔보면서 스위프트적 사고를 계속해나갈 것이다.

그때까지 안전운전하고, 그래, 조랑말들을 지켜달라! 🐴

