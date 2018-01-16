---
title: Javascript权威指南源码分析-扑克牌类
date: 2018-01-15 23:33:46
categories:
  - Javascript
tags: 
  - Javascript
  - Javascript权威指南
---
## 前言

> 书本介绍: 《Javascript权威指南 第六版》
>
> 章节：第9章 类和模块
>
> 代码案例: [9.7, 9.8]

在阅读该章节的时候，对原型原理以及衍生知识点一直似懂非懂。通过对示例代码的分析与总结，让我对类的基础知识有更形象的理解。想着记录下来供我后续阅览，于是有了这篇文章。

## 源代码

以下代码来源书本内容，由于自己重新敲了一遍，有些地方会不一致，但是不影响最终展示。

### enumeration类

```
/* 
  enumeration函数创建了一个新的枚举类，实参对象表示类的每个实例的名字和值
  返回值是一个构造函数，他标示这个新类
*/
function enumeration(namesToValues) {
  // enumeration不能作为构造函数
  const enumeration = function() {
    throw "Can't Instantiate Enumerations";
  };

  const proto = enumeration.prototype = {
    toString: function() { return this.name; },
    valueOf: function() { return this.value; },
  };

  enumeration.values = []; // 用于存放枚举对象数组
  for (let name in namesToValues) {
    // for in 会遍历对象的原型链上的所有属性，推荐使用hasOwnProperty过滤
    if (!Object.hasOwnProperty.call(namesToValues, name)) {
      continue;
    }
    const e = inherit(proto);
    e.name = name; // 设置对象name
    e.value = namesToValues[name]; // 设置对象的值
    enumeration[name] = e; // 将对象设置为构造函数的属性
    enumeration.values.push(e);
  }

  // 添加遍历方法
  enumeration.foreach = function(f, c) {
    for (let i = 0; i < this.values.length; i++) {
      f.call(c, this.values[i]);
    }
  };

  return enumeration;
}

// 继承函数
function inherit(p) {
  if (p === null) throw TypeError();
  if (Object.create) {
    return Object.create(p);
  }
  const t = typeof p;
  if (t !== "object" && t !== "function") {
    throw TypeError();
  }
  function F() {}
  F.prototype = p;
  return new F();
}
```

### poker类

```
function Poker(suit, rank) {
  this.suit = suit;
  this.rank = rank;
}

// 花色
Poker.Suit = Enumeration({
  Clubs: 1,
  Diamonds: 2,
  Hearts: 3,
  Spades: 4
});

// 牌值
Poker.Rank = Enumeration({
  Two: 2,
  Three: 3,
  Four: 4,
  Five: 5,
  Six: 6,
  Seven: 7,
  Eight: 8,
  Nine: 9,
  Ten: 10,
  Jack: 11,
  Queen: 12,
  King: 13,
  Ace: 14,
});

Poker.prototype.toString = function() {
  return `${this.rank.toString()} of ${this.suit.toString()}`;
};

// 比较牌值大小
Poker.prototype.compareTo = function(that) {
  if (this.rank < that.rank) return -1;
  if (this.rank > that.rank) return 1;
  return 0;
};

// 根据牌值计算的排序函数
Poker.orderByRank = function(a, b) {
  return a.compareTo(b);
};

// 结合花色和牌值计算的排序函数
Poker.orderBySuit = function(a, b) {
  if (a.suit < b.suit) return -1;
  if (a.suit > b.suit) return 1;
  return Poker.orderByRank(a, b);
};
```

### deck类

```
function Deck() {
  // 初始化所有的牌
  const cards = this.cards = [];
  Poker.Suit.foreach(function(s) {
    Poker.Rank.foreach(function(r) {
      cards.push(new Poker(s, r));
    });
  });
}

// 洗牌: 重新洗牌并返回洗好的牌
Deck.prototype.shuffle = function() {
  const deck = this.cards, len = deck.length;
  for (let i = len - 1; i > 0; i--) {
    let r = Math.floor(Math.random() * (i + 1));
    let temp;
    temp = deck[i];
    deck[i] = deck[r];
    deck[r] = temp;
  }
  return this;
};

// 发牌
Deck.prototype.deal = function(n) {
  if (this.cards.length < n) {
    throw "Out of cards";
  }
  return this.cards.splice(this.cards.length - n, n);
};

Deck.prototype.getValues = function() {
  return this.cards;
};
```

### 测试代码

```
const deck = (new Deck()).shuffle();
const hand = deck.deal(13).sort(Poker.orderBySuit);

// 查看全部牌
console.log('===== Deck Pockers =====');
deck.getValues().sort(Poker.orderBySuit).forEach(function(card) {
  console.log(card.toString());
});
console.log('========================\n');

// 查看手牌
console.log('===== Hand Pockers =====');
hand.forEach(function(card) {
  console.log(card.toString());
});
console.log('========================');
```

### 运行结果

```
===== Deck Pockers =====
Two of Clubs
Four of Clubs
Five of Clubs
Six of Clubs
Seven of Clubs
Eight of Clubs
Nine of Clubs
Jack of Clubs
Queen of Clubs
King of Clubs
Two of Diamonds
Three of Diamonds
Four of Diamonds
Five of Diamonds
Seven of Diamonds
Eight of Diamonds
Nine of Diamonds
Queen of Diamonds
King of Diamonds
Ace of Diamonds
Two of Hearts
Four of Hearts
Five of Hearts
Seven of Hearts
Ten of Hearts
King of Hearts
Ace of Hearts
Two of Spades
Four of Spades
Five of Spades
Six of Spades
Seven of Spades
Eight of Spades
Nine of Spades
Ten of Spades
Jack of Spades
Queen of Spades
King of Spades
Ace of Spades
========================

===== Hand Pockers =====
Three of Clubs
Ten of Clubs
Ace of Clubs
Six of Diamonds
Ten of Diamonds
Jack of Diamonds
Three of Hearts
Six of Hearts
Eight of Hearts
Nine of Hearts
Jack of Hearts
Queen of Hearts
Three of Spades
========================

```

## 其他

以上源码放在[Git](https://github.com/hpcplus2/Pocker.git)了，可直接下载运行。