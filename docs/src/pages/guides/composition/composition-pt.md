# Composição

<p class="description">Material-UI tenta tornar a composição o mais fácil possível.</p>

## Encapsulando componentes

Para fornecer o máximo de flexibilidade e desempenho, precisamos de uma maneira de conhecer a natureza dos elementos filhos que um componente recebe. To solve this problem we tag some of the components with a `muiName` static property when needed.

Você pode, no entanto, precisar encapsular um componente para melhorá-lo, que pode entrar em conflito com a solução `muiName`. Se você encapsular um componente, verifique se este tem um conjunto de propriedades estáticas.

Se você encontrar esse problema, precisará usar a mesma propriedade `muiName` do componente que será encapsulado no seu componente encapsulado. Além disso, você deve encaminhar as propriedades, já que o componente pai pode precisar controlar as propriedades do componente encapsulado.

Vamos ver um exemplo:

```jsx
const WrappedIcon = props => <Icon {...props} />;
WrappedIcon.muiName = Icon.muiName;
```

{{"demo": "pages/guides/composition/Composition.js"}}

## Propriedade component

Material-UI permite que você altere o nó raiz que será renderizado por meio de uma propriedade chamada `component`.

### Como é que funciona?

O componente será renderizado assim:

```js
return React.createElement(this.props.component, props)
```

Por exemplo, por padrão um componente `List` irá renderizar um elemento `<ul>`. Isso pode ser alterado passando um [componente React](https://reactjs.org/docs/components-and-props.html#function-and-class-components) para a propriedade `component`. O exemplo a seguir irá renderizar o componente `List` com um elemento `<nav>` como nó raiz:

```jsx
<List component="nav">
  <ListItem>
    <ListItemText primary="Trash" />
  </ListItem>
  <ListItem>
    <ListItemText primary="Spam" />
  </ListItem>
</List>
```

Esse padrão é muito poderoso e permite uma grande flexibilidade, bem como uma maneira de interoperar com outras bibliotecas, como [`react-router`](#react-router-demo) ou sua biblioteca de formulários favorita. Mas também **vem com uma pequena advertência!**

### Advertência com o uso de funções em linha

Usando uma função em linha como um argumento para a propriedade `component`, pode resultar em uma **montagem inesperada**, usando dessa forma, um novo componente será passado para a propriedade `component` toda vez que o React renderizar. Por exemplo, se você quiser cria um `ListItem` customizado que atua como link, você poderia fazer o seguinte:

```jsx
import { Link } from 'react-router-dom';

const ListItemLink = ({ icon, primary, secondary, to }) => (
  <li>
    <ListItem button component={props => <Link to={to} {...props} />}>
      {icon && <ListItemIcon>{icon}</ListItemIcon>}
      <ListItemText inset primary={primary} secondary={secondary} />
    </ListItem>
  </li>
);
```

⚠️ No entanto, como estamos usando uma função em linha para alterar o componente renderizado, o React desmontará o link toda vez que o `ListItemLink` é renderizado. Não só irá o React atualizar o DOM desnecessariamente, como o efeito cascata do `ListItem` também não funcionará corretamente.

A solução é simples: **evite funções em linha e passe um componente estático para a propriedade `component`**. Let's change the `ListItemLink` to the following:

```jsx
import { Link as RouterLink } from 'react-router-dom';

function ListItemLink(props) {
  const { icon, primary, to } = props;

  const renderLink = React.useMemo(
    () =>
      React.forwardRef((itemProps, ref) => (
        // com react-router-dom@^5.0.0 use `ref` ao invés de `innerRef`
        <RouterLink to={to} {...itemProps} innerRef={ref} />
      )),
    [to],
  );

  return (
    <li>
      <ListItem button component={renderLink}>
        <ListItemIcon>{icon}</ListItemIcon>
        <ListItemText primary={primary} />
      </ListItem>
    </li>
  );
}
```

`renderLink` agora sempre referenciará o mesmo componente.

### Advertência com abreviações

Você pode aproveitar o encaminhamento de propriedades para simplificar o código. Neste exemplo, não criamos nenhum componente intermediário:

```jsx
import { Link } from 'react-router-dom';

<ListItem button component={Link} to="/">
```

⚠️ No entanto, esta estratégia sofre de uma pequena limitação: colisão de propriedades. O componente que fornece a propriedade `component` (por exemplo, ListItem) pode não encaminhar todas as suas propriedades para o elemento raiz.

### Demonstração com React Router

Aqui está uma demonstração com [React Router DOM](https://github.com/ReactTraining/react-router):

{{"demo": "pages/guides/composition/ComponentProperty.js"}}

### Usando TypeScript

Você pode encontrar os detalhes no [guia TypeScript](/guides/typescript/#usage-of-component-property).

## Advertência com refs

Esta seção aborda advertências ao usar um componente customizado como `children` ou para a propriedade `component`.

Alguns dos componentes precisam acessar o nó DOM. Anteriormente, isso era possível usando `ReactDOM.findDOMNode`. Esta função está obsoleta em favor da utilização de `ref` e encaminhamento de ref. No entanto, apenas os seguintes tipos de componentes podem receber um `ref`:

- Qualquer componente do Material-UI
- componentes de classe, ou seja, `React.Component` ou `React.PureComponent`
- Componentes DOM (ou hospedeiro), por exemplo, `div` ou `button`
- [Componentes React.forwardRef](https://reactjs.org/docs/react-api.html#reactforwardref)
- [Componentes React.lazy](https://reactjs.org/docs/react-api.html#reactlazy)
- [Componentes React.memo](https://reactjs.org/docs/react-api.html#reactmemo)

Se você não usar um dos tipos acima ao usar seus componentes em conjunto com o Material-UI, poderá ver um aviso do React no seu console semelhante a:

> Function components cannot be given refs. Attempts to access this ref will fail. Did you mean to use React.forwardRef()?

Esteja ciente que você ainda receberá este aviso para componentes `lazy` ou `memo` se eles forem encapsulados por um componente que não contém ref.

In some instances an additional warning is issued to help with debugging, similar to:

> Invalid prop `component` supplied to `ComponentName`. Expected an element type that can hold a ref.

Only the two most common use cases are covered. Para mais informações, consulte [esta seção na documentação oficial do React](https://reactjs.org/docs/forwarding-refs.html).

```diff
- const MyButton = props => <div role="button" {...props} />;
+ const MyButton = React.forwardRef((props, ref) => <div role="button" {...props} ref={ref} />);
<Button component={MyButton} />;
```

```diff
- const SomeContent = props => <div {...props}>Olá mundo!</div>;
+ const SomeContent = React.forwardRef((props, ref) => <div {...props} ref={ref}>Olá mundo!</div>);
<Tooltip title="Hello, again."><SomeContent /></Tooltip>;
```

Para descobrir se o componente de Material-UI que você está usando tem esse requisito, verifique na documentação de propriedades da API do componente. Se você precisar encaminhar refs, a descrição será vinculada a esta seção.

### Caveat with StrictMode

If you use class components for the cases described above you will still see warnings in `React.StrictMode`. `ReactDOM.findDOMNode` is used internally for backwards compatibility. You can use `React.forwardRef` and a designated prop in your class component to forward the `ref` to a DOM component. Doing so should not trigger any more warnings related to the deprecation of `ReactDOM.findDOMNode`.

```diff
class Component extends React.Component {
  render() {
-   const { props } = this;
+   const { forwardedRef, ...props } = this.props;
    return <div {...props} ref={forwardedRef} />;
  }
}

-export default Component;
+export default React.forwardRef((props, ref) => <Component {...props} forwardedRef={ref} />);
```