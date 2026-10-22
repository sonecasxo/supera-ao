# Encontro de Superações — GitHub Pages + Firebase (sem Storage)

Esta versão funciona no GitHub Pages e usa o Firebase Realtime Database para os relatos e também para guardar as fotos compactadas. **Não usa Supabase, PHP, SQLite nem Firebase Storage**, então não precisa ativar o plano Blaze para o armazenamento de imagens.

## 1. Configurar o Firebase
1. Acesse https://console.firebase.google.com/ e abra o projeto `site`.
2. Em Configurações do projeto > Geral, registre um aplicativo Web e copie a configuração.
3. Abra `index.html` e substitua os valores do objeto `firebaseConfig` pelos dados do seu app.
4. No Authentication > Método de login, ative **Anônimo**.
5. Crie o Realtime Database. Use a URL mostrada no banco no campo `databaseURL`.
6. Não precisa ativar Storage.

## 2. Regras do Realtime Database
Na aba **Realtime Database > Regras**, substitua pelas regras abaixo e clique em **Publicar**:

```json
{
  "rules": {
    "relatos": {
      ".read": true,
      ".indexOn": ["criado_em"],
      "$relatoId": {
        ".write": "auth != null",
        ".validate": "newData.hasChildren(['titulo', 'texto', 'criado_em'])",
        "titulo": {
          ".validate": "newData.isString() && newData.val().length > 0 && newData.val().length <= 100"
        },
        "texto": {
          ".validate": "newData.isString() && newData.val().length > 0 && newData.val().length <= 5000"
        },
        "imagem_url": {
          ".validate": "!newData.exists() || (newData.isString() && newData.val().length <= 400000)"
        },
        "criado_em": {
          ".validate": "newData.isNumber()"
        },
        "$other": {
          ".validate": false
        }
      }
    }
  }
}
```

Essas regras permitem leitura pública e criação de relatos por visitantes autenticados anonimamente. O site compacta as fotos para JPEG de até 700 px antes de salvá-las no banco. Fotos muito grandes após a compactação são recusadas.

## 3. Publicar no GitHub Pages
1. Crie um repositório no GitHub.
2. Envie `index.html` e `README.md` para a raiz do repositório.
3. Abra **Settings > Pages**.
4. Em **Build and deployment**, selecione **Deploy from a branch**, escolha a branch `main` e a pasta `/ (root)`, depois salve.
5. Aguarde o endereço publicado aparecer em **Settings > Pages**.

## Importante
- Não configure regras públicas de escrita para todo o banco.
- Como os relatos são públicos e a publicação é anônima, considere adicionar moderação e proteção contra spam antes de divulgar.
- O site mostra os 100 relatos mais recentes.
- Fotos ficam dentro dos dados do Realtime Database, não em um serviço de arquivos separado. Isso é simples para um projeto pequeno, mas pode consumir a cota do banco mais rapidamente do que Storage.
