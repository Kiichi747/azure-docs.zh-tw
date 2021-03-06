---
title: "安裝並使用以 Linux 為基礎之 HDInsight (Hadoop) 上的 Giraph | Microsoft Docs"
description: "在本主題中，您將學習如何使用指令碼動作在以 Linux 為基礎的 HDInsight 叢集上安裝 Giraph。 透過變更叢集組態或自訂安裝服務與公用程式，指令碼動作可讓您在叢集建立期間自訂叢集。"
services: hdinsight
documentationcenter: 
author: Blackmist
manager: jhubbard
editor: cgronlun
tags: azure-portal
ms.assetid: 9fcac906-8f06-4002-9fe8-473e42f8fd0f
ms.service: hdinsight
ms.custom: hdinsightactive
ms.workload: big-data
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/08/2017
ms.author: larryfr
translationtype: Human Translation
ms.sourcegitcommit: 785d3a8920d48e11e80048665e9866f16c514cf7
ms.openlocfilehash: 98008946357e3c4f15f43f7901ac5ffa2dab0b60
ms.lasthandoff: 04/12/2017


---
# <a name="install-giraph-on-hdinsight-hadoop-clusters-and-use-giraph-to-process-large-scale-graphs"></a>在 HDInsight Hadoop 叢集上安裝 Giraph，以及使用 Giraph 來處理大規模圖形

您也可以使用 [指令碼動作]  來自訂叢集，在 Azure HDInsight 上 Hadoop 中的任何一種叢集上安裝 Giraph。

在本主題中，您會了解如何使用指令碼動作來安裝 Giraph。 一旦您已安裝 Giraph，您也將了解如何將 Giraph 用於大部分的一般應用程式，也就是處理大型的圖形。

> [!IMPORTANT]
> 此文件中的步驟需要使用 Linux 的 HDInsight 叢集。 Linux 是唯一使用於 HDInsight 3.4 版或更新版本的作業系統。 如需詳細資訊，請參閱 [Windows 上的 HDInsight 取代](hdinsight-component-versioning.md#hdi-version-33-nearing-deprecation-date)。

## <a name="whatis"></a>什麼是 Giraph？

[Apache Giraph](http://giraph.apache.org/) 可讓您利用 Hadoop 執行圖形處理，且可以搭配 Azure HDInsight 一起使用。 圖形可以模型化物件之間的關聯，例如大型網路 (像是網際網路) 上的路由器之間的連線，或社交網路上的人際關係 (有時稱為社交圖形)。 圖形處理可讓您分析圖形中物件之間的關聯，例如：

* 根據目前的人際關係找出可能的朋友。

* 識別網路中兩台電腦之間的最短路線。

* 計算網頁的頁面排名。

> [!WARNING]
> 透過 HDInsight 叢集提供的元件會受到完整支援，且 Microsoft 支援服務會協助釐清與解決這些元件的相關問題。
>
> 自訂元件 (例如 Giraph) 則獲得商務上合理的支援，協助您進一步對問題進行疑難排解。 如此可能會進而解決問題，或要求您利用可用管道，以找出開放原始碼技術，從中了解該技術的深度專業知識。 例如，有許多社群網站可以使用，像是：[HDInsight 的 MSDN 論壇](https://social.msdn.microsoft.com/Forums/azure/en-US/home?forum=hdinsight)、[http://stackoverflow.com](http://stackoverflow.com)。 另外，Apache 專案在 [http://apache.org](http://apache.org) 上有專案網站，例如 [Hadoop](http://hadoop.apache.org/)。


## <a name="what-the-script-does"></a>指令碼會執行哪些作業

此指令碼會執行下列動作：

* 將 Giraph 安裝至 `/usr/hdp/current/giraph`

* 將 `giraph-examples.jar` 檔案複製到您的叢集的預設儲存體 (WASB)：`/example/jars/giraph-examples.jar`

## <a name="install"></a>使用指令碼動作安裝 Giraph

在 HDInsight 叢集上安裝 Giraph 的範例指令碼位於下列位置。

    https://hdiconfigactions.blob.core.windows.net/linuxgiraphconfigactionv01/giraph-installer-v01.sh

本節提供如何在使用 Azure 入口網站建立叢集時使用範例指令碼的指示。

> [!NOTE]
> Azure PowerShell、Azure CLI、HDInsight .NET SDK 或 Azure Resource Manager 範本也可用來套用指令碼動作。 您也可以將指令碼動作套用到執行中的叢集上。 如需詳細資訊，請參閱 [使用指令碼動作自訂 HDInsight 叢集](hdinsight-hadoop-customize-cluster-linux.md)。

1. 使用 [建立以 Linux 為基礎的 HDInsight 叢集](hdinsight-hadoop-create-linux-clusters-portal.md)中的步驟開始建立叢集，但是不完成建立。

2. 在 [選用組態] 刀鋒視窗中，選取 [指令碼動作]，並提供下列資訊：

   * **名稱**：輸入指令碼動作的易記名稱。

   * **指令碼 URI**：https://hdiconfigactions.blob.core.windows.net/linuxgiraphconfigactionv01/giraph-installer-v01.sh

   * **HEAD**：勾選此選項

   * **WORKER**：讓此屬性保持未核取狀態

   * **ZOOKEEPER**：讓此屬性保持未核取狀態

   * **參數**：將此欄位保留空白

3. 在 [指令碼動作] 底部，使用 [選取] 按鈕以儲存組態。 最後，使用 [選用組態] 刀鋒視窗底部的 [選取] 按鈕，儲存選用組態資訊。

4. 繼續如 [建立以 Linux 為基礎的 HDInsight 叢集](hdinsight-hadoop-create-linux-clusters-portal.md)中所述建立叢集。

## <a name="usegiraph"></a>如何在 HDInsight 中使用 Giraph？

一旦完成建立叢集之後，請使用下列步驟以執行 Giraph 隨附的 SimpleShortestPathsComputation 範例。 這會使用基本 [Pregel](http://people.apache.org/~edwardyoon/documents/pregel.pdf) 實作，用於尋找圖表中物件之間最短的路徑。

1. 使用 SSH 連線到 HDInsight 叢集

        ssh USERNAME@CLUSTERNAME-ssh.azurehdinsight.net

    如需相關資訊，請參閱[搭配 HDInsight 使用 SSH](hdinsight-hadoop-linux-use-ssh-unix.md)。

2. 使用以下命令建立名為 **tiny_graph.txt** 的新檔案：

    ```
    nano tiny_graph.txt
    ```

    使用下列項目做為此檔案的內容：

    ```
    [0,0,[[1,1],[3,3]]]
    [1,0,[[0,1],[2,2],[3,1]]]
    [2,0,[[1,2],[4,4]]]
    [3,0,[[0,3],[1,1],[4,4]]]
    [4,0,[[3,4],[2,4]]]
    ```

    此資料使用 `[source_id, source_value,[[dest_id], [edge_value],...]]` 格式，描述有向圖形中物件之間的關聯。 每一行代表 `source_id` 物件和一或多個 `dest_id` 物件之間的關聯性。 `edge_value` (或權數) 可以視為 `source_id` 和 `dest\_id` 之間的連線強度或距離。

    如果使用值 (或權數) 當做物件之間的距離繪製出來，上述資料可能如下圖所示：

    ![tiny_graph.txt drawn as circles with lines of varying distance between](./media/hdinsight-hadoop-giraph-install-linux/giraph-graph.png)

3. 若要儲存檔案，依序使用 **Ctrl+X**、**Y**，最後按 **Enter** 以接受檔案名稱。

4. 使用下列項目以將資料儲存至您的 HDInsight 叢集的主要儲存體：

    ```
    hdfs dfs -put tiny_graph.txt /example/data/tiny_graph.txt
    ```

5. 使用下列命令執行 SimpleShortestPathsComputation 範例。

    ```
    yarn jar /usr/hdp/current/giraph/giraph-examples.jar org.apache.giraph.GiraphRunner org.apache.giraph.examples.SimpleShortestPathsComputation -ca mapred.job.tracker=headnodehost:9010 -vif org.apache.giraph.io.formats.JsonLongDoubleFloatDoubleVertexInputFormat -vip /example/data/tiny_graph.txt -vof org.apache.giraph.io.formats.IdWithValueTextOutputFormat -op /example/output/shortestpaths -w 2
    ```

    與此命令搭配使用的參數會在下表中說明：

   | 參數 | 作用 |
   | --- | --- |
   | `jar /usr/hdp/current/giraph/giraph-examples.jar` |包含範例的 jar 檔案。 |
   | `org.apache.giraph.GiraphRunner` |用於啟動範例的類別。 |
   | `org.apache.giraph.examples.SimpleShortestPathsCoputation` |所使用的範例。 在此案例中，它會計算圖形中識別碼 1 和所有其他識別碼之間的最短路徑。 |
   | `-ca mapred.job.tracker=headnodehost:9010` |叢集的前端節點。 |
   | `-vif org.apache.giraph.io.formats.JsonLongDoubleFloatDoubleVertexInputFromat` |用於輸入資料的輸入格式。 |
   | `-vip /example/data/tiny_graph.txt` |輸入資料檔案。 |
   | `-vof org.apache.giraph.io.formats.IdWithValueTextOutputFormat` |輸出格式。 在此案例中，識別碼和值是純文字。 |
   | `-op /example/output/shortestpaths` |輸出位置。 |
   | `-w 2` |要使用的背景工作角色數目。 在此案例中為 2。 |

    如需這些項目以及與 Giraph 範例搭配使用之其他參數的詳細資訊，請參閱 [Giraph 快速入門](http://giraph.apache.org/quick_start.html)。

6. 一旦作業完成，結果會儲存在 **/example/out/shotestpaths** 目錄中。 輸出檔的名稱會以 **part-m-** 開頭，結束的數字表示是第一個、第二個檔案，依此類推。 使用下列項目以檢視輸出：

    ```
    hdfs dfs -text /example/output/shortestpaths/*
    ```

    輸出應該如下所示：

        0    1.0
        4    5.0
        2    2.0
        1    0.0
        3    1.0

    SimpleShortestPathComputation 範例已刻意設計成從物件識別碼 1 開始，尋找前往其他物件的最短路徑。 因此，輸出應該會顯示 `destination_id distance`，其中 distance 是物件識別碼 1 與目標識別碼之間經過的邊緣的值 (或權數)。

    顯現為圖形後，您可以走過識別碼 1 與其他所有物件之間的最短路徑來驗證結果。 請注意，識別碼 1 和識別碼 4 之間的最短路徑為 5。 這是<span style="color:orange">識別碼 1 和 3 之間</span>加上<span style="color:red">識別碼 3 和 4 之間</span>的總距離。

    ![Drawing of objects as circles with shortest paths drawn between](./media/hdinsight-hadoop-giraph-install-linux/giraph-graph-out.png)

## <a name="next-steps"></a>後續步驟

* [在 HDInsight 叢集上安裝及使用色調](hdinsight-hadoop-hue-linux.md)。 Hue 是 Web UI，可讓您更輕鬆地建立、執行及儲存 Pig 和 Hive 工作，以及瀏覽您的 HDInsight 叢集的預設儲存體。

* [在 HDInsight 叢集上安裝 R](hdinsight-hadoop-r-scripts-linux.md)：如何使用叢集自訂，以在 HDInsight Hadoop 叢集上安裝和使用 R 的指示。 R 是一個用於統計計算的開放原始碼語言和環境。 它提供數百個內建的統計函式及它自己的程式設計語言，此語言結合了函式型和物件導向程式設計的層面。 它也提供廣泛的圖形功能。

* [在 HDInsight 叢集上安裝 Solr](hdinsight-hadoop-solr-install-linux.md)。 在 HDInsight Hadoop 叢集上使用叢集自訂安裝 Solr。 Solr 可讓您對儲存的資料執行功能強大的搜尋作業。

