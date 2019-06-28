---
layout: post
title: "Railsで複数のCSVファイルをzipに固めてダウンロードさせる"
date: "2019-06-28 10:20:34 +0900"
tags:
  - ruby
  - rails
---
# Railsで複数のCSVファイルをzipに固めてダウンロードさせる

コントローラの中で、こんな感じでやってやるとイケるはず。

```ruby
# 一括CSVダウンロード
def bulkcsv
  filename = "csv_archive.zip"
  fullpath = "#{Rails.root}/tmp/#{filename}"
  Zip::File.open(fullpath, Zip::File::CREATE) do |zipfile|
    zipfile.get_output_stream("csv/aaa.csv") do |f|
      f.puts(
        CSV.generate(encoding: Encoding::SJIS) do |csv|
          csv << %w(aaa1 aaa2 aaa3)
          csv << %w(101 102 103)
          csv << %w(201 202 203)
        end
      )
    zipfile.get_output_stream("csv/bbb.csv") do |f|
      f.puts(
        CSV.generate(encoding: Encoding::SJIS) do |csv|
          csv << %w(bbb1 bbb2 bbb3)
          csv << %w(101 102 103)
          csv << %w(201 202 203)
        end
      )
    end
  end
  # zipをダウンロードして、直後に削除する
  send_data File.read(fullpath), filename: filename, type: 'application/zip'
  File.delete(fullpath)
end
```

これで、csv_archive.zip というファイルが落っこちてくる。
その中には、csvというフォルダが一個ある。
csvフォルダの中には、aaa.csvとbbb.csvの２ファイルが格納されている。はず。はずだ。

ポイントは、 `get_output_stream` メソッド。これで物理ファイルを作らずに直接圧縮されたファイルを出力できる。
