# АНАЛИЗ ДАННЫХ И ИСКУССТВЕННЫЙ ИНТЕЛЛЕКТ [in GameDev]
Отчет по лабораторной работе #2 выполнил(а):
- Амбрушкевич Артем Антонович
- РИ-211002

Отметка о выполнении заданий (заполняется студентом):

| Задание | Выполнение | Баллы |
| ------ | ------ | ------ |
| Задание 1 | * | 60 |
| Задание 2 | * | 20 |
| Задание 3 | * | 20 |

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
- Задание 3.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Выводы.

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
### Реализовал запись в Google-таблицу набора данных, полученных с помощью линейной регрессии из лабораторной работы № 1
Написал две новых функции, первая - formating - убирает из строки '[' и ']', вторая - shupdate - добавляет на вторую страницу googlesheet необходимые данные.
Код Python-файла
```Python
import numpy as np
import matplotlib.pyplot as plt
import gspread

gc = gspread.service_account(filename='lab-3-unitydatasience-41cf8942da56.json')
sh = gc.open("UnitySheetsLab2").get_worksheet(1)

x = [3, 21, 22, 34, 54, 34, 55, 67, 89, 99]
x = np.array(x)
y = [2, 22, 24, 65, 79, 82, 55, 130, 150, 199]
y = np.array(y)

# Show the effect of a scatter plot
plt.scatter(x, y)


def model(a, b, x):
    return a * x + b


def loss_function(a, b, x, y):
    num = len(x)
    prediction = model(a, b, x)
    return (0.5 / num) * (np.square(prediction - y)).sum()


def optimize(a, b, x, y):
    num = len(x)
    prediction = model(a, b, x)

    da = (1.0 / num) * ((prediction - y) * x).sum()
    db = (1.0 / num) * ((prediction - y).sum())
    a = a - Lr * da
    b = b - Lr * db
    return a, b


def iterate(a, b, x, y, times):
    for i in range(times):
        a, b = optimize(a, b, x, y)
    return a, b

def formating(value):
    string = str(value)
    string = string.replace('[', '')
    string = string.replace(']', '')
    string = string.replace('.', ',')
    return string

i = 1
def shupdate(a, b, loss, i):
    sh.update(('A' + str(i)), formating(a))
    sh.update(('B' + str(i)), formating(b))
    sh.update(('C' + str(i)), formating(loss))


# НАЧАТЬ ИТЕРАЦИЮ

# Шаг 1. Инициализация и модель итеративной оптимизации
a = np.random.rand(1)
print(a)
b = np.random.rand(1)
print(b)
Lr = 0.000001

sh.update(('A' + str(i)), formating(a))
sh.update(('B' + str(i)), formating(b))
i += 1



a, b = iterate(a, b, x, y, 1)
prediction = model(a, b, x)
loss = loss_function(a, b, x, y)
print(a, b, loss)
plt.scatter(x, y)
plt.plot(x, prediction)

shupdate(a, b, loss, i)
i += 1


# Шаг 2. На второй итерации отображаются значения параметров, значения потерь и эффекты визуализации после итерации
a, b = iterate(a, b, x, y, 2)
prediction = model(a, b, x)
loss = loss_function(a, b, x, y)
print(a, b, loss)
plt.scatter(x, y)
plt.plot(x, prediction)

shupdate(a, b, loss, i)
i += 1




# Шаг 3. Третья итерация показывает значения параметров, значения потерь и визуализацию после итерации
a, b = iterate(a, b, x, y, 3)
prediction = model(a, b, x)
loss = loss_function(a, b, x, y)
print(a, b, loss)
plt.scatter(x, y)
plt.plot(x, prediction)

shupdate(a, b, loss, i)
i += 1


# Шаг 4. На четвертой итерации отображаются значения параметров, значения потерь и эффекты визуализации
a, b = iterate(a, b, x, y, 4)
prediction = model(a, b, x)
loss = loss_function(a, b, x, y)
print(a, b, loss)
plt.scatter(x, y)
plt.plot(x, prediction)

shupdate(a, b, loss, i)
i += 1


# Шаг 5. Пятая итерация показывает значение параметра, значение потерь и эффект визуализации после итерации
a, b = iterate(a, b, x, y, 5)
prediction = model(a, b, x)
loss = loss_function(a, b, x, y)
print(a, b, loss)
plt.scatter(x, y)
plt.plot(x, prediction)

shupdate(a, b, loss, i)
i += 1


# Шаг 6 10000-я итерация, показывающая значения параметров, потери и визуализацию после итерации
a, b = iterate(a, b, x, y, 10000)
prediction = model(a, b, x)
loss = loss_function(a, b, x, y)
print(a, b, loss)
plt.scatter(x, y)
plt.plot(x, prediction)

shupdate(a, b, loss, i)
i += 1


plt.show()
```

### Результат
   ![image](https://user-images.githubusercontent.com/97295011/194346980-e9f1f804-19f9-4236-9657-b0a334b9bacc.png)

## Задание 3
### Самостоятельно разработал сценарий воспроизведения звукового сопровождения в Unity в зависимости от изменения считанных данных из задания 2.
* создал новый скрипт и повесил его на новый объект.
Код:
```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.Networking;
using SimpleJSON;
using System.Globalization;

public class For3 : MonoBehaviour
{
    public AudioClip goodSpeak;
    public AudioClip badSpeak;
    private AudioSource selectAudio;
    private Dictionary<string, float> dataSet = new Dictionary<string, float>();
    private bool statusStart = false;
    private int i = 1;

    void Start()
    {
        StartCoroutine(GoogleSheets());
    }

    void Update()
    {
        if(dataSet["iter " + i.ToString()] > 0.36430268 & statusStart == false & i <= dataSet.Count)
        {
            StartCoroutine(PlaySelectAudioGood());
            Debug.Log(dataSet["iter " + i.ToString()]);
        }
        if(dataSet["iter " + i.ToString()] <= 0.36430268 & statusStart == false & i <= dataSet.Count)
        {
            StartCoroutine(PlaySelectAudioBad());
            Debug.Log(dataSet["iter " + i.ToString()]);
        }
    }

    IEnumerator GoogleSheets()
    {
        int j = 1;
        UnityWebRequest curentResp = UnityWebRequest.Get("https://sheets.googleapis.com/v4/spreadsheets/1I4CorKG18vvYRwGI53U-gwIgIA6PzjLE9tkd62hfTT8/values/Лист2?key=AIzaSyCcIhSeQ5BwbM2aH3HE3tWKBybRuMilaCM");
        yield return curentResp.SendWebRequest();
        string rawResp = curentResp.downloadHandler.text;
        var rawJson = JSON.Parse(rawResp);
        foreach (var itemRawJson in rawJson["values"])
        {
            var parseJson = JSON.Parse(itemRawJson.ToString());
            var selectRow = parseJson[0].AsStringList;
            selectRow = parseJson[0].AsStringList;
            dataSet["iter " + j.ToString()] = float.Parse(selectRow[1]);
            if (j < 7){j++;}
        }
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
* В результате имеем, что при значениях со 2 колонки второго листа нашей таблицы больше 0.36430268 проигрывается звук "Horosho", в ругих случаях звук "Ploho". 

![image](https://user-images.githubusercontent.com/97295011/194354341-d071b763-a303-4457-9846-78568d7c8a28.png)


## Выводы
В результате проделанной работы я познакомился с программными средствами для организции передачи данных между инструментами google, Python и Unity.
  




## Powered by

**BigDigital Team: Denisov | Fadeev | Panov**
