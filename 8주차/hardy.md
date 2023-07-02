## item 51: 의존성 분리를 위해 미러 타입 사용하기

- CSV 파일을 파싱하는 라이브러리를 만든다고 가정합니다.
- 이 라이브러리는 NodeJS 환경까지 고려해 구현합니다.

```tsx
fuction parseCSV(contents: string | Buffer): {[column: string]: string}[] {
	if(typeof contents === 'object') {
		return parseCSV(contents.toString('utf8'))
	}
}
```

- NodeJS에서 Buffer 타입을 사용하기 위해선 @types/node를 설치해야합니다.
- 하지만 @types/node를 설치하면 두 그룹의 사용자들에게 문제가 생깁니다
    - @types와 무관한 자바스크립트 개발자
    - NodeJS와 무관한 타입스크립트 개발자
- Buffer는 NodeJS 사용자만 필요하고 @types/node는 NodeJS와 타입스크립트를 동시에 사용하는 개발자만 관련이있습니다.
- 두 그룹의 사용자들은 각자가 사용하지 않는 모듈이 포함되어 있는 상황입니다.
- 이 경우 각자가 필요한 모듈만 사용할 수 있도록 구조적 타이핑을 적용할 수 있습니다.

```tsx
interface CsvBuffer {
	toString(encoding: string): string;
}
fuction parseCSV(contents: string | CsvBuffer): {[column: string]: string}[] {...}
```

- Buffer대신 필요한 메서드와 속성만 별도로 작성해 해결할 수 있습니다.

## tsc

```tsx
/* eslint-disable */
const { exec } = require("child_process");
const fs = require("fs");

function checkModuleImportError(lines) {
  /**
   * TS2307: Cannot find module
   * TS2304: Cannot find name
   */
  const errors = lines.filter(item => item.includes("TS2307") || item.includes("TS2304"));
  if (errors.length) {
    console.error(errors.join("\n"));
    throw "checkModuleImportError failed";
  }
}

function getErrorLines(data) {
  return data.split("\n").filter(item => item && item.includes("error TS"));
}

function getUniqueFilenames(lines) {
  /**
   * 아래는 tsc 에러 로그 예
   * src/stories/v3/inputs/Input.stories.tsx(83,3): error TS2322: Type 'null' is not ...
   */
  return lines
    .map(line => line.substring(0, line.indexOf("(")))
    .reduce((acc, item) => {
      if (!acc.includes(item)) {
        acc.push(item);
      }
      return acc;
    }, [])
    .sort();
}

function getErrorFilePath() {
  return `tsc/tsc-error.out`;
}

function saveErrorFile() {
  exec(`yarn run tsc`, (_, stdout) => {
    try {
      const lines = getErrorLines(stdout);
      checkModuleImportError(lines);
      const files = getUniqueFilenames(lines);
      fs.writeFileSync(getErrorFilePath(), files.join("\n"));
    } catch (error) {
      console.error("saveErrorFile failed:", error);
      if (error.includes("checkModuleImportError")) {
        console.error("There must be no import error");
      }
    }
  });
}

function checkError() {
  exec(`yarn run tsc`, (_, stdout) => {
    new Promise((resolve, reject) => {
      const lines = getErrorLines(stdout);
      checkModuleImportError(lines);

      const data = fs.readFileSync(getErrorFilePath(), "utf-8");
      const prevFiles = data.split("\n").filter(i => i);
      const files = getUniqueFilenames(lines);
      const newErrorFiles = files.filter(item => !prevFiles.includes(item));
      if (newErrorFiles.length) {
        reject({ newErrorFiles, lines });
        return;
      }
      const noErrorFiles = prevFiles.filter(item => !files.includes(item));
      resolve({ noErrorFiles });
    })
      .then(({ noErrorFiles }) => {
        if (noErrorFiles.length) {
          console.log(`congratulation! The errors have been reduced!`);
          console.log(noErrorFiles.join("\n"));
          console.log("\nplease run: yarn tsc-error-update");
        }
        console.log("success");
      })
      .catch(({ newErrorFiles, lines }) => {
        if (lines) {
          for (const line of lines) {
            if (newErrorFiles.some(item => line.includes(item))) {
              console.error(line);
            }
          }
        }
        if (newErrorFiles) {
          console.error("\nplease fix below files:\n", newErrorFiles.join("\n"));
        }
        process.exit(1);
      });
  });
}

if (process.argv[2] === "update") {
  saveErrorFile();
} else {
  checkError();
}
```
