---
title: 在 Flutter 中加入購物車的功能
tags:
    - Flutter
    - .NET
categories: App Development
photos:
    - https://trello-attachments.s3.amazonaws.com/5f088e4f109e0a21e54dcaf6/669x1375/ddc0a771de7211a226ea3f04d142add9/media_temp%283%29.jpg
---

# 用到的Package:
[[sahred_preferences]](https://pub.dev/packages/shared_preferences)


# 實作:

## 概念介紹
使用SharedPrefrence將一個 `{"ProductItemId": Quantity}` 的 `Map<String,int>` 利用 `json.encode()` 和 `json.decode()` 來存取在Local. 當需要使用的時候再取得所有的Key來Request所有的商品來顯示.

## 用到的Class
**Product**是用來存取通用的產品資料, 例如品名和產品介紹; **ProductItem** 則是用來存取不同的品項, 例如Size和Color.

- Product
- ProductItem

```dart
class Product{
  String id;
  String name;
  String description;
  
  // Image urls for showing the product.
  List<String> imageUrls;

  // Variants of the product. (ex: Different colors) 
  List<ProductItem> productItems;
}

class ProductItem {
  String id;
  Product belongProduct;
  String itemName;
  double price;
  int totalQuantity;
}

// With these two class, we can create...

Product newCloth = Product(
    name: "T-Shirt"
    description: "Highest quality with lowest price you can get."
    imageUrls: [
        "www.abc.com/12345.png",
        "www.abc.com/67890.png"
    ],
    productItems: [
        ProductItem(
            itemName: "Yellow"
            price: 14.5,
            totalQuantity: 50,
        ),
        ProductItem(
            itemName: "Black"
            price: 17.9,
            totalQuantity: 20,
        ),
    ]
);

```

## 功能實作
實作的功能並不複雜, 大部分只是類似經典的CRUD, 只是施作地點是在客戶端。


### 取得(Getter)和設置(Setter)購物車(使用SharedPreference)

#### 設置(Set)
```dart
 Future<bool> setShoppingCartMap(
    SharedPreferences prefs,
    Map<String, int> shoppingCartMap,
  ) async {
    bool success = await prefs.setString(
        Env.orderShoppingCartSharedPreferenceStoringString,
        json.encode(shoppingCartMap));

    if (!success)
      return throw Exception(
       "Shopping can't be saved",
      );
    return true;
  }
```
#### 取得(Get)
```dart
  Map<String, int> getShoppingCartMap(SharedPreferences prefs) {
    String productItemsString =
        prefs.getString(Env.orderShoppingCartSharedPreferenceStoringString) ??
            "{}";
    Map<String, int> shoppingCartMap = Map<String, int>.from(
      json.decode(productItemsString),
    );
    return shoppingCartMap;
  }
```
有這兩個最基本與購物車互動的方法後, 我們可以用這兩個基本的方法去建造CRUD.

### 將ProductItem加入購物車
```dart
Future<Map<String, int>> addProductItemToShoppingCart(
    ProductItem productItemToAdd,
    int quantity,
  ) async {
    final prefs = await SharedPreferences.getInstance();
    Map shoppingCartMap = getShoppingCartMap(prefs);

    // Checking if it contian the productItem has already in the shopping cart..
    bool alreadyInShoppingCart =
        shoppingCartMap.containsKey(productItemToAdd.id);

    if (alreadyInShoppingCart) {
      shoppingCartMap[productItemToAdd.id] += quantity;
    } else {
      shoppingCartMap[productItemToAdd.id] = quantity;
    }

    // Store back to prefs;
    bool saveSuccessfully = await setShoppingCartMap(prefs, shoppingCartMap);
    return saveSuccessfully ? shoppingCartMap : null;
  }
```

### 在購物車內修改數量(Quantity)
```dart
  Future<Map<String, int>> updateProductItemQuantityInShoppingCart(
    String productItemIdToRemove,
    int quantity,
  ) async {
    final prefs = await SharedPreferences.getInstance();
    Map shoppingCartMap = getShoppingCartMap(prefs);

    // Checking if it contian the productItem already.
    bool inShoppingCart = shoppingCartMap.containsKey(productItemIdToRemove);
    if (!inShoppingCart)
      //  "Product item is not in the shopping cart. Add product item to shopping cart first"
      throw Exception(
        "Item is not in the shopping cart."
      );

    if (quantity > 0) {
      shoppingCartMap[productItemIdToRemove] = quantity;
    } else {
      // Removing the product item from shopping cart if given value is negative.; 
      shoppingCartMap.remove(productItemIdToRemove);
    }

    // Store back to pref;
    bool saveSuccessfully = await setShoppingCartMap(prefs, shoppingCartMap);
    return saveSuccessfully ? shoppingCartMap : null;
  }
```

### 刪除單項ProductItem
我們可以利用上個Function來執行這項指令
```dart
  Future<Map<String, int>> removeProudctItemFromShoppingCart(
    String productItemIdToRemove,
  ) async {
    return await updateProductItemQuantityInShoppingCart(
        productItemIdToRemove, -1);
  }
```

### 清空購物車
直接將Shopping cart這個Map抹除掉.

```dart
  Future<bool> emptyShoppingCart() async {
    final prefs = await SharedPreferences.getInstance();
    return await prefs.remove(shoppingCartSotringString);
  }
```

### 顯示購物車
要顯示購物車 我們需要一個新的Class來存 
- ProductItem
- Quantity

```dart
  class ShoppingCartResult {
    Map<String, int> shoppingCartMap;
    Map<String, ProductItem> productItemsMap;

    ShoppingCartResult({
      @required this.shoppingCartMap,
      @required this.productItemsMap,
    });

    factory ShoppingCartResult.fromProductItemList(
        Map<String, int> shoppingCartMap, List<ProductItem> productItemList) {
      return ShoppingCartResult(
        shoppingCartMap: shoppingCartMap,
        productItemsMap: Map.fromIterable(
          shoppingCartMap.keys.map(
            (k) => {
              k: productItemList.firstWhere((i) => i.id == k),
            },
          ),
        ),
      );
    }
  }

  Future<ShoppingCartResult> getProductItemsInShoppingCart(
    BuildContext context,
  ) async {
    final prefs = await SharedPreferences.getInstance();
    Map<String, int> shoppingCartMap = getShoppingCartMap(prefs);

    ShoppingCartResult result = ShoppingCartResult.fromProductItemList(
      shoppingCartMap,
      shoppingCartMap?.keys?.toList()?.isNotEmpty == true
            // Functions to retrieve the productItems from server.
          ? await context
              .read<ProductService>()
              .getProductItemsFromListOfItemIds(
                context,
                shoppingCartMap.keys.toList(),
              )
          : [],
    );
    return result;
  }
```

# 結語
這個購物車實作非常簡單, 不用去實作太多server-side的部分, 但當使用者在不同裝置中登入的時時候換看見不同的購物車(無法存儲在雲端). 想要改進這個問題可以在後端連結MongoDB(或其他NoSQL)來當作緩存存取這些購物車的資料, 因購物車可能高頻率的被更新而且不需要使用太多的Join查找, 且大多數的時候只單純使用Key來搜尋. 
