# АНАЛИЗ ДАННЫХ И ИСКУССТВЕННЫЙ ИНТЕЛЛЕКТ [in GameDev]
Отчет по лабораторной работе #2 выполнил(а):
- Амбрушкевич Артем Антонович
- РИ-211002

Отметка о выполнении заданий (заполняется студентом):

| Задание | Выполнение | Баллы |
| ------ | ------ | ------ |
| Задание 1 | * | 60 |
| Задание 2 | # | 20 |
| Задание 3 | # | 20 |

знак "*" - задание выполнено; знак "#" - задание не выполнено;

Работу проверили:
- к.т.н., доцент Денисов Д.В.
- к.э.н., доцент Панов М.А.
- ст. преп., Фадеев В.О.

[![N|Solid](https://cldup.com/dTxpPi9lDf.thumb.png)](https://nodesource.com/products/nsolid)

[![Build Status](https://travis-ci.org/joemccann/dillinger.svg?branch=master)](https://travis-ci.org/joemccann/dillinger)

Структура отчета

- Данные о работе: название работы, фио, группа, выполненные задания.
- Цель работы.
- Задание 1.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Задание 2.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).

- Выводы.
- ✨Magic ✨

## Цель работы
Познакомиться с программными средствами для организции передачи данных между инструментами google, Python и Unity

## Задание 1
### Реализовал совместную работу и передачу данных в связке Python - Google-Sheets – Unity. При выполнении задания использовал видео-материалы и исходные данные, предоставленные преподавателем курса.
Ход работы:
  1. В облачном сервисе google console подключил API для работы с google sheets и google drive.
  2. Реализовал запись данных из скрипта на python в google-таблицу. Данные описывают изменение темпа инфляции на протяжении 11 отсчётных периодов, с учётом стоимости игрового объекта в каждый период.
    ![image](https://user-images.githubusercontent.com/97295011/194254398-a2f6308c-80b4-41b8-a6aa-324d2e0fe229.png)
    ![image](https://user-images.githubusercontent.com/97295011/194254723-2087220e-3349-4402-841f-d8c41546901f.png)

  3. Создал новый проект на Unity, который будет получать данные из google-таблицы, в которую были записаны данные в предыдущем пункте.
        * добавил исходные данные преподавателя курса
        
          скрипты
          
          ![image](https://user-images.githubusercontent.com/97295011/194257955-4a7c0923-9b0a-4512-ac9f-b158497242d8.png)
          
          аудио-файлы
          
          ![image](https://user-images.githubusercontent.com/97295011/194258124-0f091366-2abd-4653-8559-0e23443aef6b.png)
       
       * создал скрипт, который будет получать данные из google-таблицы и воспроизводить звук в зависимости от полученного значения
       
          ```c#
          using System.Collections;
          using System.Collections.Generic;
          using UnityEngine;
          using UnityEngine.Networking;
          using SimpleJSON;

          public class NewBehaviourScript : MonoBehaviour
          {
            public AudioClip goodSpeak;
            public AudioClip normalSpeak;
            public AudioClip badSpeak;
            private AudioSource selectAudio;
            private Dictionary<string, float> dataSet = new Dictionary<string, float>();
            private bool statusStart = false;
            private int i = 1;


            // Start is called before the first frame update
            void Start()
            {
                StartCoroutine(GoogleSheets());
            }

            // Update is called once per frame
            void Update()
            {
                if (dataSet["Mon_" + i.ToString()] <= 10 & statusStart == false & i != dataSet.Count)
                {
                    StartCoroutine(PlaySelectAudioGood());
                    Debug.Log(dataSet["Mon_" + i.ToString()]);
                }
                if (dataSet["Mon_" + i.ToString()] > 10 & dataSet["Mon_" + i.ToString()] < 100 & statusStart == false & i != dataSet.Count)
                {
                    StartCoroutine(PlaySelectAudioNormal());
                    Debug.Log(dataSet["Mon_" + i.ToString()]);
                }
                if (dataSet["Mon_" + i.ToString()] >= 100 & statusStart == false & i != dataSet.Count)
                {
                    StartCoroutine(PlaySelectAudioBad());
                    Debug.Log(dataSet["Mon_" + i.ToString()]);
                }
            }

            IEnumerator GoogleSheets()
            {
                UnityWebRequest curentResp = UnityWebRequest.Get("https://sheets.googleapis.com/v4/spreadsheets/1I4CorKG18vvYRwGI53U-gwIgIA6PzjLE9tkd62hfTT8/values/Лист1?key=AIzaSyCcIhSeQ5BwbM2aH3HE3tWKBybRuMilaCM");
                yield return curentResp.SendWebRequest();
                string rawResp = curentResp.downloadHandler.text;
                var rawJson = JSON.Parse(rawResp);
                foreach (var itemRawJson in rawJson["values"])
                {
                    var parseJson = JSON.Parse(itemRawJson.ToString());
                    var selectRow = parseJson[0].AsStringList;
                    dataSet.Add(("Mon_" + selectRow[0]), float.Parse(selectRow[2]));
                }
            //Debug.Log(dataSet["Mon_1"]);
            }
            IEnumerator PlaySelectAudioGood()
            {
                statusStart = true;
                selectAudio = GetComponent<AudioSource>();
                selectAudio.clip = goodSpeak;
                selectAudio.Play();
                yield return new WaitForSeconds(3);
                statusStart = false;
                i++;
            }
            IEnumerator PlaySelectAudioNormal()
            {
                statusStart = true;
                selectAudio = GetComponent<AudioSource>();
                selectAudio.clip = normalSpeak;
                selectAudio.Play();
                yield return new WaitForSeconds(3);
                statusStart = false;
                i++;
            }
            IEnumerator PlaySelectAudioBad()
            {
                statusStart = true;
                selectAudio = GetComponent<AudioSource>();
                selectAudio.clip = badSpeak;
                selectAudio.Play();
                yield return new WaitForSeconds(4);
                statusStart = false;
                i++;
            }
          }
          ```
          
        * Создал пустой объект GameObject, накинул на него компонент "Audio Source" и предыдущий скрипт, перетащил звуки из папки в соответствующие поля скрипта 
          ![image](https://user-images.githubusercontent.com/97295011/194262325-b4ad7890-d571-4347-a82e-61a7ec7fa1cc.png)
          
        * при запуске игры в консоль выводится "дебаг" информация и с принятием каждого значения из google-таблицы воспроизводится соответствующий звук
          ![image](https://user-images.githubusercontent.com/97295011/194263085-6b881263-942a-4223-b71d-842f4642bf07.png)

        
## Задание 2


## Выводы
  

Абзац умных слов о том, что было сделано и что было узнано.

| Plugin | README |
| ------ | ------ |
| GitHub | [plugins/github/README.md][PlGh] |


## Powered by

**BigDigital Team: Denisov | Fadeev | Panov**
