# SOLID 设计原则实践

[toc]

本篇文章将用实例说明在 react 中设计模式五大原则（SOLID）的应用方式。

[toc]

首先回顾一下 SOLID 原则

- S：Single-Responsibility principle，单一职责原则（SRP）。一个函数（组件）只负责一件事，比如要么处理迭代要么处理渲染
- O：Open-Closed principle，开放封闭原则（OCP）。对函数（组件）对扩展开放，对修改封闭
- L：Likov Substitution principle，里氏替换原则（LSP）。派生组件要能替换基类组件，所以派生组件要能接收基类组件的所有特性
- I：Interface Segregation principle，接口隔离原则（ISP）。只给函数（组件）传递他们需要的数据而不是整个接口，防止增加组件的复杂性
- D：Dependency Inversion principle，依赖倒置原则（DIP）。高层组件不依赖低层组件，而是都依赖于他们的抽象，用抽象来限制实体

## Single-Resonsibility Principle

单一职责原则要求我们的函数（组件）只有一个清晰的职责。

下面是一个错误的示例：

```react
//  ❌ Bad Practice：组件有负责迭代和渲染UI两个职责
const Prodocuts = () => {
  return (
  	<div className='products'>
    	{
        products.map(product => {
          <div key={product?.id} className='product'>
          	<h3>{product?.name}</h3>
            <p>{product?.price}</p>
          </div>
        })
      }
    </div>
  )
}
```

上面的示例中，Products 组件既负责了对 products 数组进行迭代，又负责了对 product 的渲染，拥有了多个职责，不利于后续修改。

**为了符合 SRP 我们应该这样：**

```react
//  ✅ Good Practice: 拆分职责到更小的组件，将渲染交给 Product 组件
import Product from './Product';
import products from '../../data/products.json';

const Products = () => {
    return (
        <div className="products">
            {products.map((product) => (
                <Product key={product?.id} product={product}
            ))}
        </div>
    );
};

// Product.js
const Product = ({ product }) => {
    return (
        <div className="product">
            <h3>{product?.name}</h3>
            <p>${product?.price}</p>
        </div>
    );
};
```

## Open-Closed Principle

开放封闭原则强调组件应该对扩展开放，对修改封闭。这一原则鼓励创建具有弹性、模块化和易于维护的代码。

下面是一个错误的示例：

```react
// ❌ Bad Practice: 直接在原组件的基础上修改，添加功能

// Button.js
const Button = ({ text, onClick }) => {
  return (
    <button onClick={onClick}>
      {text}
    </button>
  );
}

// Button.js
// 直接修改原组件，添加额外的参数
const Button = ({ text, onClick, icon }) => {
  return (
    <button onClick={onClick}>
      <i className={icon} />
      <span>{text}</span>
    </button>
  );
}

// Home.js
const Home = () => {
  const handleClick= () => {};

  return (
    <div>
      {/* ❌ Avoid this */}
      <Button text="Submit" onClick={handleClick} icon="fas fa-arrow-right" /> 
    </div>
  );
}
```

**为了符合 OCP 我们应该这样：**

```react
// ✅ Good Practice: 不修改原组件，而是创建一个新的特性组件来支持新的功能

// Button.js
// Existing Button functional component
const Button = ({ text, onClick }) => {
  return (
    <button onClick={onClick}>
      {text}
    </button>
  );
}

// IconButton.js
// IconButton component
// 新的组件可以依赖于老的组件，如果不好处理的话可以不依赖老的直接新写一个。比如下面这个要修改到组件里面，不好传递，所以直接重写
const IconButton = ({ text, icon, onClick }) => {
  return (
    <button onClick={onClick}>
      <i className={icon} />
      <span>{text}</span>
    </button>
  );
}

const Home = () => {
  const handleClick = () => {
    // Handle button click event
  }

  return (
    <div>
      <Button text="Submit" onClick={handleClick} />
      {/* 
      <IconButton text="Submit" icon="fas fa-heart" onClick={handleClick} />
      */}
    </div>
  );
}
```

这样处理后，我们没有对老的组件进行修改，负责 OCP 原则。OCP 原则也强调扩展函数（组件）而不是修改他们！

## Liskov Substitution Principle

里氏替换原则是面向对象编程的基本原则，强调在层次结构中对象的可替换性。推广到 react 中，我们需要派生组件能够在不影响程序的情况下替换掉基类组件。

下面是一个错误的示例：

```react
//  ❌ Bad Practice: 自定义的下拉选择组件没有支持原生的 Select 组件的所有属性，比如 multiple、disabled
// 无法完成对原生组件的替换
const BadCustomSelect = ({ value, iconClassName, handleChange }) => {
  return (
    <div>
      <i className={iconClassName}></i>
      <select value={value} onChange={handleChange}>
        <options value={1}>One</options>
        <options value={2}>Two</options>
        <options value={3}>Three</options>
      </select>
    </div>
  );
};

const LiskovSubstitutionPrinciple = () => {
  const [value, setValue] = useState(1);

  const handleChange = (event) => {
    setValue(event.target.value);
  };

  return (
    <div>
      {/** ❌ Avoid this */}
      <BadCustomSelect value={value} handleChange={handleChange} />
    </div>
  );
```

**为了符合 LIP 我们应该这样：**

```react
// ✅ Good Practice
// 通过 ...props 接收用户传过来的所有属性，传递到 select 基类上

const CustomSelect = ({ value, iconClassName, handleChange, ...props }) => {
  return (
    <div>
      <i className={iconClassName}></i>
      <select value={value} onChange={handleChange} {...props}>
        <options value={1}>One</options>
        <options value={2}>Two</options>
        <options value={3}>Three</options>
      </select>
    </div>
  );
};

const LiskovSubstitutionPrinciple = () => {
  const [value, setValue] = useState(1);

  const handleChange = (event) => {
    setValue(event.target.value);
  };

  return (
    <div>
      <CustomSelect
        value={value}
        handleChange={handleChange}
        defaultValue={1}
      />
    </div>
  );
};
```

通过修改，我们可以使用自定义的派生下拉选择组件去替换基类 select 组件了。

## Interface Segregation Principle

接口隔离原则建议我们应该专注于组件的特定要求而不是大水漫灌，迫使用户实现不必要的方法。推广到 react 中我们应该避免传递非必要的参数给组件，以增加组件的复杂性！

下面是一个错误的示例：

```react
// ❌ Avoid: 传递非必要的信息给组件
const ProductThumbnailURL = ({ product }) => {
  return (
    <div>
      <img src={product.imageURL} alt={product.name} />
    </div>
  );
};

// ❌ Bad Practice
const Product = ({ product }) => {
  return (
    <div>
      <ProductThumbnailURL product={product} />
      <h4>{product?.name}</h4>
      <p>{product?.description}</p>
      <p>{product?.price}</p>
    </div>
  );
};

const Products = () => {
  return (
    <div>
      {products.map((product) => (
        <Product key={product.id} product={product} />
      ))}
    </div>
  );
}
```

在上面的例子中，我们将整个 product 对象都传递给了ProductThumbnailURL 组件，实际上它用不到这么多属性，反而可能会给这个组件带来复杂性。

**为了符合 ISP 我们应该这样：**

```react
// ✅ Good: 减少参数，只传递需要的属性给组件
const ProductThumbnailURL = ({ imageURL, alt }) => {
  return (
    <div>
      <img src={imageURL} alt={alt} />
    </div>
  );
};

// ✅ Good Practice
const Product = ({ product }) => {
  return (
    <div>
      <ProductThumbnailURL imageURL={product.imageURL} alt={product.name} />
      <h4>{product?.name}</h4>
      <p>{product?.description}</p>
      <p>{product?.price}</p>
    </div>
  );
};

const Products = () => {
  return (
    <div>
      {products.map((product) => (
        <Product key={product.id} product={product} />
      ))}
    </div>
  );
};
```

## Dependency Inversion Principle

依赖倒置原则强调实体应该依赖于抽象而不是具体实现。在 raect 中我们的高级组件不应该依赖于低级组件。

下面是一个错误的示例：

```react
// ❌ Bad Practice：组件的实际功能（提交方法）依赖于组件自身的实现，耦合性很高，不利于维护

const BadCustomForm = ({ children }) => {
  const handleSubmit = () => {
    // submit operations
  };
  return <form onSubmit={handleSubmit}>{children}</form>;
};

const DependencyInversionPrinciple = () => {
  const [email, setEmail] = useState();

  const handleChange = (event) => {
    setEmail(event.target.value);
  };

  return (
    <div>
      {/** ❌ Avoid: 高耦合*/}
      <BadCustomForm>
        <input
          type="email"
          value={email}
          onChange={handleChange}
          name="email"
        />
      </BadCustomForm>
    </div>
  );
```

**为了符合 DIP 我们应该这样：**

```react
// ✅ Good Practice：组件的功能不再由低级组件自己实现，而是通过高级组件实现并传入

const AbstractForm = ({ children, onSubmit }) => {
  const handleSubmit = (event) => {
    event.preventDefault();
    onSubmit();
  };

  return <form onSubmit={handleSubmit}>{children}</form>;
};

const DependencyInversionPrinciple = () => {
  const [email, setEmail] = useState();

  const handleChange = (event) => {
    setEmail(event.target.value);
  };

  const handleFormSubmit = () => {
    // submit business logic hereho
  };

  return (
    <div>
      <AbstractForm onSubmit={handleFormSubmit}>
        <input
          type="email"
          value={email}
          onChange={handleChange}
          name="email"
        />
        <button type="submit">Submit</button>
      </AbstractForm>
    </div>
  );
}
```

## 总结

SOLID 原则在开发中可以作为一个指导性的原则。利用好 SOLID 原则，可以帮助我们编写出灵活的、模块化的、易维护的代码。

## 

## 参考文章

[zMastering SOLID Principles Like the Back of Your Hand in Just 8 Minutes! ](https://hackernoon.com/mastering-solid-principles-like-the-back-of-your-hand-in-just-8-minutes?source=rss) 
