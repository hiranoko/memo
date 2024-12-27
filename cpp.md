# c++


```c++
#include <vector>      // 動的配列（std::vector）を使うためのヘッダ
#include <numeric>     // std::transformなどのアルゴリズムが含まれるヘッダ
#include <execution>   // 並列化やSIMDを可能にするポリシー（std::execution）を使用するためのヘッダ

// 関数: 2つのベクトルa, bを足し合わせて、結果をベクトルcに格納する
// 入力: const std::vector<double>& a, const std::vector<double>& b
// 出力: std::vector<double>& c
void vector_add(const std::vector<double>& a, const std::vector<double>& b, std::vector<double>& c) {
    /*
    引数の説明:
    - const std::vector<double>& a: ベクトルaを参照渡し（&）して、コピーを避けて高速化。constで読み取り専用にする。
    - const std::vector<double>& b: 同上、ベクトルbも参照渡ししてコピーを回避。
    - std::vector<double>& c: 出力用のベクトルcも参照渡し。変更可能にするためconstを付けない。
    */

    std::transform(
        std::execution::par_unseq,  // 並列実行ポリシー（par_unseq: 並列化+SIMDを使用）
        a.begin(), a.end(),         // aの先頭と末尾のイテレータ（処理範囲）
        b.begin(),                  // bの対応する要素の先頭イテレータ
        c.begin(),                  // cの出力先の先頭イテレータ
        [](double x, double y) {    // ラムダ式: x, yを足して返す
            return x + y;
        }
    );
    /*
    - std::execution::par_unseq:
      - 並列化（CPUの複数コアを利用）とSIMD（1命令で複数データを処理）を有効化。
      - 並列化により大きなデータサイズで性能向上が期待できる。
    - std::transform:
      - 入力の範囲[a.begin(), a.end())の各要素をbの対応する要素と処理し、
        結果をcに格納する。
    */
}

int main() {
    // 動的配列std::vectorを100万要素で初期化。初期値は全て0。
    std::vector<double> a(1000000), b(1000000), c(1000000);

    // ランダムデータの生成
    std::generate(a.begin(), a.end(), std::rand); // aにランダム値を格納
    std::generate(b.begin(), b.end(), std::rand); // bにランダム値を格納

    // 2つのベクトルを加算する関数を呼び出し、結果をcに格納
    vector_add(a, b, c);

    return 0; // 正常終了
}
```