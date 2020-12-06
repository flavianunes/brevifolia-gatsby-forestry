## Como implementar um switch de tema dark/light no Gatsby com Styled Components

![Screenshot da tela do gatsby-starter-blog com um tema dark aplicado e um switch que faz o toggle entre ele e o tema light](https://cdn.hashnode.com/res/hashnode/image/upload/v1607264766379/Yx_Xj-fcK.png)

Permitir a utilização de um tema escuro (dark mode) no seu blog tem alguns benefícios, como **economia de bateria** e uma leitura mais confortável em ambientes escuros. Neste tutorial nós vamos aprender como deixar a pessoa usuária escolher entre um tema escuro e claro, utilizando Gatsby e Styled Components. 

# Implementação

Aqui vamos usar o `gatsby-starter-blog`, mas você pode utilizar outro starter ou seguir o processo em um projeto Gatsby já existente. Caso o seu starter ou projeto não esteja utilizando styled components, nós vamos precisar instalá-lo no projeto. [Clique aqui para acessar a documentação caso queira saber mais sobre o processo](https://www.gatsbyjs.com/docs/styled-components/).


1. Primeiro, vamos instalar os pacotes: `gatsby-plugin-styled-components` e `styled-components babel-plugin-styled-components`. No meu caso, estou utilizando o yarn como gerenciador de pacotes, então a minha instalação será feita pelo comando abaixo.
```shell
yarn add gatsby-plugin-styled-components styled-components babel-plugin-styled-components
```

2. Agora vamos adicionar o plugin `gatsby-plugin-styled-components` no array de plugins que se encontra no arquivo de configuração do Gatsby (`gatsby-config.js`).
```javascript
module.exports = {
  plugins: [`gatsby-plugin-styled-components`, ...],
}
```

3. Teste o funcionamento do styled componentes em uma página de exemplo. [A documentação te mostra como fazer isso](https://www.gatsbyjs.com/docs/styled-components/).

## Implementação

Vamos dividir a implementação do dark mode em três etapas:

* Theme, onde vamos declarar as cores na versão dark e light.
* GlobalStyle, onde vamos adicionar os estilos globais do projeto.
* Toggle, componente responsável por permitir ao usuário mudar entre um tema e outro.

### Theme

Vamos criar nosso `theme.js` na nossa pasta `components`. Nesse arquivo nós vamos exportar dois objetos: o *darkTheme* e o *lightTheme*. Se o seu projeto tem mais temas, você pode adicioná-los aqui. Coloque as cores da sua preferência e *não se esqueça de checar o contraste entre elas*! O meu ficou assim:

```javascript
const white = "#FFFFFF"
const emojiYellow = "#FEE133"
const darkBlue = "#011936"
const almostBlack = "#17161A"

export const lightTheme = {
    body: white,
    text: almostBlack,
    toggleDetail: emojiYellow,
    toggleBackground: white,
}

export const darkTheme = {
    body: darkBlue,
    text: white,
    toggleDetail: white,
    toggleBackground: emojiYellow,
}

```

### GlobalStyle

Crie um componente chamado GlobalStyle para definir os estilos globais do seu projeto. As propriedades dos seus temas vão variar muito com o seu layout. Se você ainda não tem o seu layout em mente, você pode começar com algo simples como `background-color` e `text`, variando de preto pra branco no `text` e de branco pra preto no `background-color`.

```javascript 
import { createGlobalStyle} from "styled-components"

export const GlobalStyles = createGlobalStyle`
  body {
    background-color: ${({ theme }) => theme.body};
    color: ${({ theme }) => theme.text};
  }
`
```

### Toggle

Agora nós precisamos de um componente para que a pessoa usuária alternar entre esses dois temas criados. Pra isso vamos precisar criar um componente `toggle` e alterar o nosso componente de `layout`.

Esse componente pode ser de diversos tipos. Por exemplo, se você tem três ou mais temas, você pode usar um `dropdown`. No meu caso, com dois temas, vou utilizar uma `checkbox`.

1. Crie o componente chamado Toggle e o estilize da forma que quiser. O meu está com o estilo de um `switch`. Note que o componente recebe o tema atual (`Theme`) e uma função responsável por alterar o tema (`toggleTheme`) sempre que o estado da checkbox mudar (`onChange`).
```javascript
import React from "react"
import styled from "styled-components"
const Wrapper = styled.div`
    position: relative;
`
const Label = styled.label`
  position: absolute;
  top: 0;
  left: 0;
  width: 52px;
  height: 26px;
  background-color: ${({ theme }) => theme.toggleBackground};
  box-shadow: 1px 2px 4px rgba(0, 0, 0, 0.25);
  border-radius: 38px;
  cursor: pointer;
  &::after {
    content: "";
    display: block;
    border-radius: 50%;
    width: 18px;
    height: 18px;
    margin: 3px;
    background-color: ${({ theme }) => theme.toggleDetail};
    box-shadow: 1px 2px 4px rgba(0, 0, 0, 0.25);
    transition: 0.2s;
  }
`;
const CheckBox = styled.input`
  opacity: 0;
  z-index: 1;
  border-radius: 38px;
  width: 52px;
  height: 26px;
  &:checked + ${Label} {
    background-color: ${({ theme }) => theme.toggleBackground};
    &::after {
      background-color: ${({ theme }) => theme.toggleDetail};
      margin-left: 26px;
    }
  }
`;
const Toggle = ({theme,  toggleTheme }) => {
    return (
          <Wrapper>
            <CheckBox id="checkbox" type="checkbox" onChange={toggleTheme}/>
            <Label htmlFor="checkbox" />
          </Wrapper>
    )
}
export default Toggle
```

2. Agora nós vamos precisar alterar o componente Layout. Primeiro, importe tudo que for necessário, como os seus temas, o seu estilo global, o componente Toggle e o useState e useEffects do React, já que *vamos usar hooks pra gerenciar o estado do tema*. 
O `themeMode` mostrado abaixo vai associar o valor do seu `theme` (light ou dark) com o tema que você está importando (lightTheme e darkTheme), para que possamos passar o tema como props para o `ThemeProvider` e o componente `Toggle`. 
O método `themeToggler` altera o tema atual para o seu oposto, ou seja: se o tema ativo é o light, o `themeToggler` altera o theme para dark.
```javascript
import React, { useState, useEffect } from "react"
import { ThemeProvider } from "styled-components"
import { lightTheme, darkTheme } from "./theme"
import GlobalStyles from "./global"
import Toggle from "./toggle"
...
const Layout = ({ location, title, children }) => {
  ...
  const [theme, setTheme] = useState("light")
  const themeMode = theme === 'light' ? lightTheme : darkTheme

  const themeToggler = () => {
    theme === "light" ? setTheme("dark") : setTheme("light")
  }
  ...
  return (
    <ThemeProvider theme={themeMode}>
      <GlobalStyles/>
      <Toggle theme={themeMode} toggleTheme={themeToggler}/>
      ...
    </ThemeProvider>
  )
}
export default Layout
```
# Resultado

[Você pode ver como ficou o resultado aqui](http://dark-mode-styled-components.surge.sh) e pode [conferir o código fonte desse projeto aqui](https://github.com/flavianunes/dark-mode-styled-components) .