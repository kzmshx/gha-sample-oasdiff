# gha-sample-oasdiff

[Tufin/oasdiff](https://github.com/Tufin/oasdiff) を使って GitHub Action で OpenAPI Specification の差分を取得し、レビューコメントを投稿するサンプル。

## 1. まずローカルで oasdiff を使ってみる

[OAI/OpenAPI-Specification](https://github.com/OAI/OpenAPI-Specification) に OpenAPI Specification の例があり、差分を確認できるものとして

- examples/v3.0/petstore.yaml ([Raw](https://github.com/OAI/OpenAPI-Specification/raw/main/examples/v3.0/petstore.yaml))
- examples/v3.0/petstore-expanded.yaml ([Raw](https://raw.githubusercontent.com/OAI/OpenAPI-Specification/main/examples/v3.0/petstore-expanded.yaml))

があるので、まずはこれらを使って差分の抽出を試す。

CLI をインストールし、

```shell
# https://github.com/Tufin/oasdiff#install-on-macos-with-brew
brew tap tufin/oasdiff
brew install oasdiff
```

`petstore.yaml` と `petstore-expanded.yaml` の差分を取得する。

```shell
oasdiff \
  -base https://raw.githubusercontent.com/OAI/OpenAPI-Specification/main/examples/v3.0/petstore.yaml \
  -revision https://raw.githubusercontent.com/OAI/OpenAPI-Specification/main/examples/v3.0/petstore-expanded.yaml \
  -format text
```

<details>
<summary>出力</summary>
<div>

```md
### New Endpoints: 1
--------------------
DELETE /pets/{petId}

### Deleted Endpoints: None
---------------------------

### Modified Endpoints: 3
-------------------------
GET /pets
- Description changed from '' to 'Returns all pets from the system that the user has access to
Nam sed condimentum est. Maecenas tempor sagittis sapien, nec rhoncus sem sagittis sit amet. Aenean at gravida augue, ac iaculis sem. Curabitur odio lorem, ornare eget elementum nec, cursus id lectus. Duis mi turpis, pulvinar ac eros ac, tincidunt varius justo. In hac habitasse platea dictumst. Integer at adipiscing ante, a sagittis ligula. Aenean pharetra tempor ante molestie imperdiet. Vivamus id aliquam diam. Cras quis velit non tortor eleifend sagittis. Praesent at enim pharetra urna volutpat venenatis eget eget mauris. In eleifend fermentum facilisis. Praesent enim enim, gravida ac sodales sed, placerat id erat. Suspendisse lacus dolor, consectetur non augue vel, vehicula interdum libero. Morbi euismod sagittis libero sed lacinia.

Sed tempus felis lobortis leo pulvinar rutrum. Nam mattis velit nisl, eu condimentum ligula luctus nec. Phasellus semper velit eget aliquet faucibus. In a mattis elit. Phasellus vel urna viverra, condimentum lorem id, rhoncus nibh. Ut pellentesque posuere elementum. Sed a varius odio. Morbi rhoncus ligula libero, vel eleifend nunc tristique vitae. Fusce et sem dui. Aenean nec scelerisque tortor. Fusce malesuada accumsan magna vel tempus. Quisque mollis felis eu dolor tristique, sit amet auctor felis gravida. Sed libero lorem, molestie sed nisl in, accumsan tempor nisi. Fusce sollicitudin massa ut lacinia mattis. Sed vel eleifend lorem. Pellentesque vitae felis pretium, pulvinar elit eu, euismod sapien.
'
- New query param: tags
- Modified query param: limit
  - Description changed from 'How many items to return at one time (max 100)' to 'maximum number of results to return'
  - Schema changed
    - Max changed from 100 to null
- Responses changed
  - Modified response: 200
    - Description changed from 'A paged array of pets' to 'pet response'
    - Content changed
      - Modified media type: application/json
        - Schema changed
          - MaxItems changed from 100 to null
          - Items changed
            - Property 'AllOf' changed
              - 2 schemas added
            - Type changed from 'object' to ''
            - Required changed
              - Deleted required property: id
              - Deleted required property: name
            - Properties changed
              - Deleted property: id
              - Deleted property: name
              - Deleted property: tag
    - Headers changed
      - Deleted header: x-next

POST /pets
- Description changed from '' to 'Creates a new pet in the store. Duplicates are allowed'
- Request body changed
- Responses changed
  - New response: 200
  - Deleted response: 201

GET /pets/{petId}
- Description changed from '' to 'Returns a user based on a single ID, if the user does not have access to the pet'
- Modified path param: petId
  - Description changed from 'The id of the pet to retrieve' to 'ID of pet to fetch'
  - Schema changed
    - Type changed from 'string' to 'integer'
    - Format changed from '' to 'int64'
- Responses changed
  - Modified response: 200
    - Description changed from 'Expected response to a valid request' to 'pet response'
    - Content changed
      - Modified media type: application/json
        - Schema changed
          - Property 'AllOf' changed
            - 2 schemas added
          - Type changed from 'object' to ''
          - Required changed
            - Deleted required property: id
            - Deleted required property: name
          - Properties changed
            - Deleted property: id
            - Deleted property: name
            - Deleted property: tag

Servers changed
- New server: https://petstore.swagger.io/v2
- Deleted server: http://petstore.swagger.io/v1
```

</div>
</details>

oasdiff は `-check-breaking` オプションを指定すると、API 仕様の変更に含まれる破壊的変更を抽出してくれる。

```shell
oasdiff \
  -base https://raw.githubusercontent.com/OAI/OpenAPI-Specification/main/examples/v3.0/petstore.yaml \
  -revision https://raw.githubusercontent.com/OAI/OpenAPI-Specification/main/examples/v3.0/petstore-expanded.yaml \
  -check-breaking \
  -format text
```

<details>
<summary>出力</summary>
<div>

```text
Backward compatibility errors (7):
error	[added-required-request-body] at https://github.com/OAI/OpenAPI-Specification/raw/main/examples/v3.0/petstore-expanded.yaml
	in API POST /pets
		added required request body

error	[response-success-status-removed] at https://github.com/OAI/OpenAPI-Specification/raw/main/examples/v3.0/petstore-expanded.yaml
	in API POST /pets
		removed the success response with the status '201'

error	[request-parameter-type-changed] at https://github.com/OAI/OpenAPI-Specification/raw/main/examples/v3.0/petstore-expanded.yaml
	in API GET /pets/{petId}
		for the 'path' request parameter 'petId', the type/format was changed from 'string'/'none' to 'integer'/'int64'

error	[response-body-type-changed] at https://github.com/OAI/OpenAPI-Specification/raw/main/examples/v3.0/petstore-expanded.yaml
	in API GET /pets/{petId}
		the response's body type/format changed from 'object'/'none' to 'none'/'none' for status '200'

error	[response-required-property-removed] at https://github.com/OAI/OpenAPI-Specification/raw/main/examples/v3.0/petstore-expanded.yaml
	in API GET /pets/{petId}
		removed the required property 'id' from the response with the '200' status

error	[response-required-property-removed] at https://github.com/OAI/OpenAPI-Specification/raw/main/examples/v3.0/petstore-expanded.yaml
	in API GET /pets/{petId}
		removed the required property 'name' from the response with the '200' status

warning	[optional-response-header-removed] at https://github.com/OAI/OpenAPI-Specification/raw/main/examples/v3.0/petstore-expanded.yaml
	in API GET /pets
		the optional response header 'x-next' removed for the status '200'
```

</div>
</details>

JSON で出力することもできる。

```shell
oasdiff \
  -base https://raw.githubusercontent.com/OAI/OpenAPI-Specification/main/examples/v3.0/petstore.yaml \
  -revision https://raw.githubusercontent.com/OAI/OpenAPI-Specification/main/examples/v3.0/petstore-expanded.yaml \
  -check-breaking \
  -format json
```

<details>
<summary>出力</summary>
<div>

```json
[
  {
    "id": "added-required-request-body",
    "text": "added required request body",
    "level": 0,
    "operation": "POST",
    "path": "/pets",
    "source": "https://github.com/OAI/OpenAPI-Specification/raw/main/examples/v3.0/petstore-expanded.yaml"
  },
  {
    "id": "response-success-status-removed",
    "text": "removed the success response with the status '201'",
    "level": 0,
    "operation": "POST",
    "path": "/pets",
    "source": "https://github.com/OAI/OpenAPI-Specification/raw/main/examples/v3.0/petstore-expanded.yaml"
  },
  {
    "id": "request-parameter-type-changed",
    "text": "for the 'path' request parameter 'petId', the type/format was changed from 'string'/'none' to 'integer'/'int64'",
    "level": 0,
    "operation": "GET",
    "path": "/pets/{petId}",
    "source": "https://github.com/OAI/OpenAPI-Specification/raw/main/examples/v3.0/petstore-expanded.yaml"
  },
  {
    "id": "response-body-type-changed",
    "text": "the response's body type/format changed from 'object'/'none' to 'none'/'none' for status '200'",
    "level": 0,
    "operation": "GET",
    "path": "/pets/{petId}",
    "source": "https://github.com/OAI/OpenAPI-Specification/raw/main/examples/v3.0/petstore-expanded.yaml"
  },
  {
    "id": "response-required-property-removed",
    "text": "removed the required property 'id' from the response with the '200' status",
    "level": 0,
    "operation": "GET",
    "path": "/pets/{petId}",
    "source": "https://github.com/OAI/OpenAPI-Specification/raw/main/examples/v3.0/petstore-expanded.yaml"
  },
  {
    "id": "response-required-property-removed",
    "text": "removed the required property 'name' from the response with the '200' status",
    "level": 0,
    "operation": "GET",
    "path": "/pets/{petId}",
    "source": "https://github.com/OAI/OpenAPI-Specification/raw/main/examples/v3.0/petstore-expanded.yaml"
  },
  {
    "id": "optional-response-header-removed",
    "text": "the optional response header 'x-next' removed for the status '200'",
    "level": 1,
    "operation": "GET",
    "path": "/pets",
    "source": "https://github.com/OAI/OpenAPI-Specification/raw/main/examples/v3.0/petstore-expanded.yaml"
  }
]
```

</div>
</details>

## 2. oasdiff-action を使ってみようと思ったが Deprecated だった

oasdiff の開発元が GitHub Action で oasdiff を実行する [oasdiff/oasdiff-action](https://github.com/oasdiff/oasdiff-action) のリポジトリを公開している。
しかし、マーケットプレイスには該当のリポジトリに紐づいたアクションは存在せず、代わりに Deprecated になったリポジトリのほうのアクション（[openapi-spec-diff](https://github.com/marketplace/actions/openapi-spec-diff)）が残っていた。
これを使うのも微妙なので、自前でワークフローを作ってみる。

## 3. oasdiff を GitHub Action で実行する

まず、単に oasdiff を実行するだけのワークフローを作る。
oasdiff の開発元が Docker Hub 上に oasdiff の [Docker Image](https://hub.docker.com/r/tufin/oasdiff) を公開しているので、これを使う。

```yaml
jobs:
  oasdiff-diff:
    runs-on: ubuntu-latest
    container:
      image: tufin/oasdiff:stable
    steps:
      - uses: actions/checkout@v3
      - run: |
          oasdiff \
            -base https://github.com/OAI/OpenAPI-Specification/raw/main/examples/v3.0/petstore.yaml \
            -revision https://github.com/OAI/OpenAPI-Specification/raw/main/examples/v3.0/petstore-expanded.yaml \
            -format text
```

## 4. oasdiff の結果をレビューコメントとして出力する

このままでは差分取得結果を確認するために actions/runs を見に行く必要があり不便なので、oasdiff の結果をレビューコメントとして出力するようにする。

```yaml
...
      - run: |
          oasdiff \
            -base https://github.com/OAI/OpenAPI-Specification/raw/main/examples/v3.0/petstore.yaml \
            -revision https://github.com/OAI/OpenAPI-Specification/raw/main/examples/v3.0/petstore-expanded.yaml \
            -format text >result.txt

          echo "OASDIFF_DIFF_RESULT<<EOF" >> $GITHUB_ENV
          echo "$(cat result.txt)" >> $GITHUB_ENV
          echo "EOF" >> $GITHUB_ENV
        id: oasdiff-diff
      - uses: actions/github-script@v6
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          script: |
            const result = process.env.OASDIFF_DIFF_RESULT;
            const issue_number = context.issue.number;
            const owner = context.repo.owner;
            const repo = context.repo.repo;

            const bodyPrefix = `<!-- oasdiff-diff-result -->`;
            const body = `${bodyPrefix}
            ## Changes
            ${result}`;

            const { data: comments } = await github.rest.issues.listComments({ issue_number, owner, repo });
            const comment_id = comments.find((c) => c.body.startsWith(bodyPrefix))?.id;
            if (comment_id) {
              await github.rest.issues.updateComment({ comment_id, issue_number, owner, repo, body });
            } else {
              await github.rest.issues.createComment({ issue_number, owner, repo, body });
            }
```
