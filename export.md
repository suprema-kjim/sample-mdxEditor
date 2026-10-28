# PDF로 내보내기

PDF는 `book.yml`에 정의한 문서를 순서대로 합쳐 만듭니다. 커버와 백 커버, 목차는 템플릿에서 가져옵니다.

## book.yml 작성하기

| 항목 | 설명 | 예 |
| --- | --- | --- |
| `title` | 문서 제목. 커버와 PDF 문서 정보에 쓰입니다. | MDX Editor |
| `subtitle` | 부제목(문서 형식). 커버에 쓰입니다. | 사용자 가이드 |
| `version` | 문서 버전 | 0.1.0 |
| `lang` | 문서 언어 코드. 목차 제목, 커버의 이름표, 언어별 템플릿 선택에 쓰입니다. | ko-KR |
| `docuNum` | 문서 번호 | 1.00 |
| `outputName` | 출력 파일 이름 | MDX_Editor_User_Guide |
| `template` | 사용할 템플릿. 경로 또는 템플릿 이름을 씁니다. | templates/sample |
| `documents` | 본문에 들어갈 문서 목록. 이 순서대로 합칩니다. | introduction.md |

## 템플릿 고르기

템플릿은 폴더 하나입니다. 폴더에는 `template.json` 설정 파일과 커버, 백 커버 HTML, 본문 CSS가 들어 있습니다. 새 프로젝트를 만들면 선택한 템플릿이 프로젝트의 `templates` 폴더에 복사되므로, 이 프로젝트에 맞게 자유롭게 고칠 수 있습니다.

`book.yml`의 `template` 항목에 적은 템플릿은 다음 순서로 찾습니다.

1. 프로젝트 안의 경로 (`templates/sample`)
2. 여러 프로젝트가 함께 쓰는 사용자 템플릿 폴더
3. 에디터에 내장된 템플릿

```json [templates/sample/template.json]
{
  "name": "Sample",
  "version": "0.1.0",
  "styles": ["default.css"],
  "cover": "cover.html",
  "back": "back.html",
  "tocLevels": [1, 2],
  "bookmarkLevels": [1, 2]
}
```

문서 언어에 맞는 `{언어 코드}-back.html` 파일이 있으면 그 파일을 백 커버로 씁니다. 이 문서는 한국어이므로 `ko-back.html`을 씁니다.

### 커버에 book.yml 값 넣기

커버와 백 커버 HTML에서 `{{title}}`, `{{subtitle}}`, `{{version}}` 같은 자리 표시자는 `book.yml`의 값으로 바뀝니다. 위 표에 없는 속성도 `book.yml`에 자유롭게 추가하고 같은 이름으로 쓸 수 있습니다.

```yaml [book.yml]
author: 기술 문서팀
product:
  name: MDX Editor
  edition: Community
```

```html [cover.html]
<p class="author">{{author}}</p>
<p class="product">{{product.name}} {{product.edition}}</p>
```

- `{{product.name}}`처럼 점으로 안쪽 속성을 씁니다. 목록은 쉼표로 이어서 넣습니다.
- `{{version}}`(버전 0.1.0), `{{lang}}`(한국어), `{{docuNum}}`은 보기 좋게 다듬은 값입니다. 적은 그대로의 값은 `{{book.version}}`처럼 `book.`을 붙여 씁니다.
- `book.yml`에 없는 이름은 빈칸이 됩니다. 그래서 속성이 서로 다른 여러 book이 템플릿 하나를 함께 쓸 수 있습니다.

### 커버의 속성 목록 늘리기

`properties`에 `이름표: 값`을 적으면 템플릿을 고치지 않아도 HTML 미리보기 커버의 속성 목록에 적은 순서대로 들어갑니다. 값을 비우면 그 줄은 넣지 않습니다.

```yaml [book.yml]
properties:
  작성자: MDX Editor 팀
  라이선스: MIT
```

템플릿에서는 `<dl>` 안에 `{{properties}}`를 두면 `<dt>이름표</dt><dd>값</dd>` 목록이 들어갑니다. 값 하나만 쓸 때는 `{{properties.작성자}}`처럼 씁니다.

## 출력하기

1. `book-render`로 HTML과 매니페스트를 만듭니다.
2. `pdf-engine`으로 매니페스트를 PDF로 출력합니다.

```bash [터미널]
cd packages/book-render
npx tsx src/cli.ts build <프로젝트>/book.yml --out <프로젝트>/out
cd ../../crates/pdf-engine
cargo run -- <프로젝트>/out/manifest.json
```

출력된 PDF를 열면 왼쪽에 북마크 창이 열려 있고, 목차의 항목을 클릭하면 해당 쪽으로 이동합니다.
