# c++

## 使いそうなライブラリ

| ヘッダファイル / ライブラリ | 主な内容・機能                                                               |
| --------------------------- | ---------------------------------------------------------------------------- |
| `<iostream>`                | 標準入出力（`cin`, `cout`, `cerr` など）                                     |
| `<vector>`                  | 動的配列（`std::vector`）                                                    |
| `<array>`                   | 固定長配列（`std::array`）                                                   |
| `<string>`                  | 文字列操作（`std::string`）                                                  |
| `<algorithm>`               | ソート・検索・変換などのアルゴリズム（`std::sort`, `std::find` など）        |
| `<numeric>`                 | 数値演算（`std::accumulate`, `std::inner_product` など）                     |
| `<map>`                     | キーと値のペアを管理（`std::map`, `std::unordered_map`）                     |
| `<set>`                     | 集合の管理（`std::set`, `std::unordered_set`）                               |
| `<deque>`                   | 両端キュー（`std::deque`）                                                   |
| `<list>`                    | 双方向リスト（`std::list`）                                                  |
| `<stack>`                   | スタックデータ構造（`std::stack`）                                           |
| `<queue>`                   | キューと優先度付きキュー（`std::queue`, `std::priority_queue`）              |
| `<bitset>`                  | ビット操作を扱う（`std::bitset`）                                            |
| `<cmath>`                   | 数学関数（`std::sqrt`, `std::pow`, `std::sin` など）                         |
| `<random>`                  | 乱数生成（`std::mt19937`, `std::uniform_int_distribution` など）             |
| `<thread>`                  | マルチスレッドプログラミング（`std::thread`, `std::mutex` など）             |
| `<execution>`               | 並列処理ポリシー（`std::execution::par`, `std::execution::par_unseq`）       |
| `<fstream>`                 | ファイル入出力（`std::ifstream`, `std::ofstream`）                           |
| `<chrono>`                  | 時間計測と操作（`std::chrono::steady_clock`, `std::this_thread::sleep_for`） |
| `<tuple>`                   | 複数の値を保持するタプル（`std::tuple`）                                     |
| `<utility>`                 | ユーティリティ関数（`std::pair`, `std::move`, `std::swap` など）             |
| `<opencv2/opencv.hpp>`      | OpenCVの全体ヘッダ（画像処理・コンピュータビジョン）                         |
| `<opencv2/highgui.hpp>`     | 画像表示ウィンドウ管理（`cv::imshow`, `cv::waitKey` など）                   |
| `<opencv2/imgproc.hpp>`     | 画像処理（フィルタリング、変換、特徴抽出など）                               |
| `<opencv2/core.hpp>`        | 基本構造（`cv::Mat`, `cv::Point` など）                                      |
| `<opencv2/imgcodecs.hpp>`   | 画像の入出力（`cv::imread`, `cv::imwrite` など）                             |
| `<opencv2/videoio.hpp>`     | 動画の入出力（`cv::VideoCapture`, `cv::VideoWriter` など）                   |
| `Boost`                     | 高度なデータ構造やアルゴリズム、数学、マルチスレッドなどの拡張ライブラリ     |
| `Eigen`                     | 線形代数ライブラリ（行列演算、数値解析など）                                 |
| `Cereal`                    | C++の直列化ライブラリ（オブジェ                                              |

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