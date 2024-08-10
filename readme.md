@wymjs/merge-env
===

> 使用 Typescript 來編寫環境變量

## 安裝

```shell
# lodash-es, esbuild 都是依賴庫
$ pnpm i @wymjs/merge-env lodash-es
$ pnpm i -D esbuild
```


## 使用

```typescript
// 這段可以參考 test/env/，這段從 test/env/.env.ts 貼進來的
// 可以看到環境變數可以隨便寫邏輯，用來生成動態環境時很方便
import { execSync } from 'child_process'
import { helloWorld } from '../helper'

function version() {
  const commitId = execSync('git rev-parse HEAD')?.toString()
  return `v_${commitId.substring(0, 8)}`
}

const envConfig = {
  port: 9247,
  version: version(),
  hello: helloWorld(),
  // 測試合併的參數
  merge: {
    dir: 'common',
    name: '.env.ts',
  },
}

// 將通用類型使用 typeof 轉成類型，給其他環境引入使用
// 這樣就不用手動申明類型了，其他環境也不用再寫這段
export type EnvType = typeof envConfig
// 用來讓自己知道，自己有配置哪些環境的環境變數
export type EnvMode = 'development' | 'production'
// 重要! 一定要用 export default 導出，因為內部使用 .default 取出來做合併
export default envConfig



// 使用
import path from 'path'
import { mergeEnv } from '@wymjs/merge-env'

const envConfig2 = await mergeEnv<EnvType, EnvMode>({
  // mode 為當前環境值
  mode: 'production',
  // dirs 為環境變數的存放目錄，絕對路徑且可多個，後蓋前
  dirs: [
    path.resolve(process.cwd(), 'env/common'), 
    path.resolve(process.cwd(), 'env/last')
  ],
})

// 這將會是 production 環境下合併過後的最終環境變數，合併規則都 vite 如下：
// .env -> .env.local -> .env.[mode] -> .env.[mode].local
// 以下用 common 代替上面的 env/common, last 代替 env/last
// 上到下為取出順序，下個 merge 上個的方式合併數據，
// 以下使用上面定義的 mode production 來描述最多會合成的檔案會是以下：
//              common .env
// (向 ↑ merge) common .env.local
// (向 ↑ merge) common .env.procution
// (向 ↑ merge) common .env.procution.local
// (向 ↑ merge) last .env
// (向 ↑ merge) last .env.local
// (向 ↑ merge) last .env.procution
// (向 ↑ merge) last .env.procution.local
console.log(envConfig2)
```
