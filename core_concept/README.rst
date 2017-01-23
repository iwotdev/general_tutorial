.. contents::

基本概念
========

服務架構
--------

|1|

身份驗證和授權
--------------

iWoT Cloud Service 支援 SSL/TLS。TLS 使用 `X.509 <https://zh.wikipedia.org/wiki/X.509>`_ 認證，接著利用非對稱加密演算來對通訊方做身分認證，之後交換對稱金鑰作為會談金鑰（Session key）。這個會談金鑰是用來將通訊兩方交換的資料做加密，保證兩個應用間通訊的保密性和可靠性，使客戶與伺服器應用之間的通訊不被攻擊者竊聽。

Message Borker
--------------

Message Broker 支援 MQTT、WebSocket。使用發行/訂閱模型來交換訊息，以進行一對一和一對多雙向通訊。透過這種通訊模式，iWoT Cloud Service 能夠讓連接的裝置將某個特定主題的資料廣播發送給多個訂閱者。

Device Shadow 與 Log
--------------------

iWoT Cloud Server 會保留一份最新的裝置狀態，即使裝置離線，應用程式也可以取得裝置最後一次同步的狀態，並且修改或觸發裝置的動作。當裝置上線時，iWoT Cloud Server 會通知裝置執行動作，並且同步裝置狀態。

iWoT Cloud Server 同時會記錄所有的訊息，使用者可以查詢歷史記錄和統計訊息用量。

規則引擎
--------

iWoT Cloud Service 的規則引擎提供使用者圖型介面，建立自定義的商業規則，處理傳入 iWoT 的訊息，並將訊息轉換並傳輸到其他的裝置或雲端服務。

規則引擎分為全域規則引擎和裝置閘道規則引擎。當裝置閘道離線時，裝置閘道規則引擎仍然可以執行閘道內的裝置的商業規則。

訊息流程
--------

裝置觸發事件或是狀態改變。

|2|

應用程式觸發規則。

|3|

應用程式改變裝置狀態或是觸發動作。

|4|

Thing Model
===========

Model 架構
-----------

|5|

id：用來識別裝置的 ID，必須是唯一值。id 是必要欄位。

classID：用來識別裝置類型的 ID，必須是唯一值，且不可以和 id 相同。classID 是必要欄位。

name：裝置的名稱，主要的用途是讓使用者識別裝置。可以是任意的字串。name 是選填的欄位。

description：裝置的詳細描述。description 是選填的欄位。

tags：裝置的標簽，一個字串的陣列，用來標示裝置的特徵。tags 是選填的欄位。

customFields：使用者自定的 JSON Object，可以包含數個鍵值(key-value pair)。customFields 是選填的欄位。

actions：描述裝置可被觸發的動作，必需包含一個 id，如果動作需要有參數，可在 values 物件裏存放多個參數物件。請參閱 *Value Object*。

properties：描述裝置的屬性，必需包含一個 id，一個屬性可以包含多個屬性值，存放在 values 物件裏。請參閱 *Value Object*。

events：描述裝置的事件，必需包含一個id，如果事件需要有參數，可在"values"物件裏存放多個參數物件。請參閱 *Value Object*。

::

    {
      "id": <String>,
      "name": <String>,
      "description": <String>,
      "values": {
        "valueID_1": <Value Object>,
        "valueID_2": <Value Object>,
        ...
      }
    }

system：描述裝置的系統資訊，必需包含一個 firmwareVersion。system 是選填的欄位。

Value Object：

-  valueID：參數值的 ID，必需在 values 物件中唯一的 ID。
-  name：參數值的名稱，主要的用途是讓使用者識別參數值，可以是任意的字串。name 是選填的欄位。
-  description：參數值的詳細描述。description 是選填的欄位。
-  type：參數值的型態，支援的資料型態為：integer, float, boolean, string, enum。type 是選填的欄位，如果有填寫參數型態，iWoT 會驗證參數值型態。
-  unit：參數值的單位，主要的用途是讓使用者識別參數值的單位。unit 是選填的欄位。
-  require：參數值是否為必要欄位。require 是選填欄位，如果有填寫，iWoT 會驗證訊息是否包含必要參數值。
-  minValue：參數值的最小值。minValue 是選填欄位，如果有填寫，iWoT 會驗證參數值是否符合限制。
-  maxValue：參數值的最大值。maxValue 是選填欄位，如果有填寫，iWoT 會驗證參數值是否符合限制。

::

    {
      ...
      "values": {
        "<valueName>": {
          "name": <String>,
          "description": <String>,
          "type": <String>,
          "unit": <String>,
          "required": <Boolean>,
          "minValue": <Numeric>,
          "maxValue": <Numeric>
        },
        ...
      }
      ...
    }

內建 Actions
----------------

iWoT 系統內建三個系統服務使用的 Actions，使用者不可以定義相同名稱的 action。

**upgradeFirmware: iWoT用於發送韌體更新動作。**

::

    {
        "upgradeFirmware": {
            "name": "Upgrade Device Firmware",
            "description": "Loads a new firmware from the cloud and installs it.",
            "values": {
                "delay": {
                    "name": "Upgrade Delay",
                    "type": "integer",
                    "required": true,
                    "minValue": 0,
                    "maxValue": 120,
                    "unit": "seconds"
                },
                "url": {
                    "name": "Firmware URL",
                    "description": "The URL to get the firmware from (should contain the credentials).",
                    "required": true,
                    "type": "string"
                },
                "requester": {
                    "name": "Request User ID",
                    "description": "Your user ID (optional) ",
                    "type": "string"
                },
                "version": {
                    "name": "User defined version",
                    "description": "The version of user upload firmware.",
                    "type": "string"
                },
                "modelVersion": {
                    "name": "Model version",
                    "description": "The model version.",
                    "type": "string"
                },
                "codeVersion": {
                    "name": "Code version",
                    "description": "The blocky code version.",
                    "type": "string"
                },
                "firmwareVersion": {
                    "name": "Firmware version",
                    "description": "The firmware version.",
                    "type": "string"
                }
            }
        }
    }

**createTunnel: 用於建立裝置閘道規則引擎的通訊。**

::

    {
        "createTunnel": {
            "name": "create ssh tunnel",
            "description": "Prepare ssh tunnel to server",
            "values": {
                "user": {
                    "name": "server reserved user for tunnel",
                    "type": "string",
                    "required": true
                },
                "passwd": {
                    "name": "passwd of server reserved user for tunnel",
                    "type": "string",
                    "required": true
                },
                "ip": {
                    "name": "server public ip",
                    "type": "string",
                    "required": true
                },
                "port": {
                    "name": "server reserved port",
                    "type": "integer",
                    "required": true
                }
            }
        }
    }

**closeTunnel: 用於關閉裝置閘道規則引擎的通訊。**

::

    {
        "closeTunnel": {
            "name": "close ssh tunnel ",
            "description": "Release tunnel binding resource",
            "values": {}
        }
    }

範例
----

基本溫度感應器的 Web Thing Model：

::

    {
        "id": "sampleThine",
        "classID": "sampleThine_Class",
        "properties": {
            "temperature": {
                "values": {
                    "temp": {
                        "type": "float"
                    }
                }
            }
        },
        "system": {
            "firmwareVersion": "1.0"
        }
    }

完整的例子：

::

    {
        "id": "sampleThine",
        "classID": "sampleThine_Class",
        "name": "Shopping Cart",
        "description": "Shopping Cart that updates its lock status and environment information",
        "tags": [
            "cart",
            "device",
            "sample"
        ],
        "customFields": {
            "size": "20",
            "color": "blue"
        },
        "actions": {
            "actions": {
                "lock": {
                    "name": "Lock shopping cart",
                    "value": {
                        "lockAction": {
                            "enum": {
                                "LOCK": "Lock the shopping cart",
                                "UNLOCK": "Unlock the shopping cart"
                            }
                        }
                    }
                }
            }
        },
        "events": {
            "deviceError": {
                "name": "Device Error"
            },
            "batteryLow": {
                "name": "Low Battery",
                "description": "Triggered once when battery drops below 2.8v.",
                "values": {
                    "state": {
                        "enum": {
                            "REPLACE": "It's time to replace the battery.",
                            "CRITICAL": "This device will die anytime soon"
                        }
                    }
                },
                "customFields": {
                    "status": "active"
                }
            },
            "myCustomEventType": {
                "name": "My Event"
            }
        },
        "properties": {
            "sample": {
                "name": "this is a test",
                "values": {
                    "int1": {
                        "name": "integer normal",
                        "type": "integer",
                        "minValue": 0,
                        "maxValue": 120
                    },
                    "int2": {
                        "name": "integer without min",
                        "type": "integer",
                        "maxValue": 120
                    },
                    "int3": {
                        "name": "integer without max",
                        "type": "integer",
                        "minValue": 50
                    },
                    "int4": {
                        "name": "integer without limit",
                        "type": "integer"
                    },
                    "float1": {
                        "name": "float normal",
                        "type": "float",
                        "minValue": "24.5",
                        "maxValue": "42.3"
                    },
                    "float2": {
                        "name": "float without min",
                        "type": "float",
                        "maxValue": "42.3"
                    },
                    "float3": {
                        "name": "float without max",
                        "type": "float",
                        "minValue": "24.5"
                    },
                    "boolean1": {
                        "name": "boolean",
                        "type": "boolean"
                    },
                    "string1": {
                        "name": "string",
                        "type": "string"
                    },
                    "enum1": {
                        "name": "enum",
                        "enum": {
                            "opt1": "optiont1",
                            "opt2": "option2"
                        }
                    }
                }
            },
            "temperature": {
                "name": "Temperature Sensor",
                "description": "An ambient temperature sensor.",
                "values": {
                    "temp": {
                        "name": "Temperature sensor",
                        "description": "The temperature in celsius",
                        "unit": "celsius",
                        "customFields": {
                            "gpio": 21
                        }
                    }
                },
                "tags": [
                    "sensor",
                    "public",
                    "indoors"
                ]
            },
            "humidity": {
                "name": "Humidity Sensor",
                "description": "An ambient humidity sensor.",
                "values": {
                    "h": {
                        "name": "Humidity",
                        "description": "Percentage of Humidity",
                        "unit": "percent",
                        "customFields": {
                            "gpio": 21
                        }
                    }
                },
                "tags": [
                    "sensor",
                    "public"
                ]
            }
        },
        "system": {
            "firmwareVersion": "1.0",
            "upSince": "2015-10-31T23:59:59.000Z",
            "_connections": {
                "ip": "198.39.3.2",
                "port": 8585,
                "rootUrl": "http://iwot.io/",
                "publicUrl": "http://iwot.io/sampleThine"
            },
            "_identifiers": {
                "serialNumber": "AX2332-00021",
                "ean": "9399392392"
            },
            "_myCustomMeta1": {},
            "_myCustomMeta2": {}
        }
    }

Web Thing Model
---------------

`https://www.w3.org/Submission/2015/SUBM-wot-model-20150824/ <https://www.w3.org/Submission/2015/SUBM-wot-model-20150824/>`__

.. |1| image:: https://raw.githubusercontent.com/iwotdev/general_tutorial/master/core_concept/images/1.png
.. |2| image:: https://raw.githubusercontent.com/iwotdev/general_tutorial/master/core_concept/images/2.png
.. |3| image:: https://raw.githubusercontent.com/iwotdev/general_tutorial/master/core_concept/images/3.png
.. |4| image:: https://raw.githubusercontent.com/iwotdev/general_tutorial/master/core_concept/images/4.png
.. |5| image:: https://raw.githubusercontent.com/iwotdev/general_tutorial/master/core_concept/images/5.png
