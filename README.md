# Jenkins shared library for 1C:Enterprise 8

## Цель

Создание библиотеки (или плагина) для Jenkins, позволяющей:

* максимально упростить написание Jenkinsfile для процесса CI в условиях платформы 1С:Предприятие 8
* иметь схожий и контролируемый пайплайн для всех проектов
* дать пользователю в руки простой декларативный конфигурационный файл, вместо требования описывать всю сложную логику по работе с 1С 

## Общие положения

* в активной разработке и поиске "своего пути" по разработке библиотеки;
* формат конфигурационного файла **не** стабилизирован;
* обратная совместимость **пока** не гарантируется, внимательно читайте changelog;
* количество stage будет со временем увеличиваться;
* использовать на свой страх и риск;
* любая помощь приветствуется.

## Ограничения

1. Для шага подготовки требуется любой агент с меткой `agent`.
1. Для запуска шага анализа SonarQube требуется агент с меткой `sonar`.
1. Для запуска шага валидации EDT требуется агент с меткой `edt` (для собственно валидации) и агент с меткой `oscript` (для трансформации результатов с помощью библиотеки [stebi](https://github.com/Stepa86/stebi)).
1. Для запуска шагов, работающих с 1С (подготовка, синтаксический контроль и т.д.) требуется агент с меткой, совпадающей со значением в поле `v8version` файла конфигурации.
1. В качестве ИБ используется файловая база, создаваемая в `./build/ib`, без данных авторизации. Переопределение "в следующих сериях".
1. Stage "Дымовые тесты" пока пустой.
1. Запуск `vrunner` на текущий момент происходит из локального каталога `oscript_modules`. Предполагается наличие в корне репозитория файла `packagedef`, в котором бы была указана зависимость от `vanessa-runner`

## Возможности

1. Все шаги можно запустить на базе docker-образов из форка репозитория onec-docker. См. [памятку по слоям и последовательности сборки](https://github.com/firstBitSemenovskaya/onec-docker/blob/feature/first-bit/Layers.md)
1. Трансформация кода из формата конфигуратора в формат EDT (только если включен шаг `edtValidate`).
1. Подготовка информационной базы по версии из хранилища конфигурации.
1. Запуск синтаксического контроля средствами конфигуратора и сохранение результатов в виде отчета jUnit.
1. Запуск валидации проекта средствами EDT и конвертация отчета в формате generic issues.
1. Запуск статического анализа для SonarQube

## Подключение

Инструкция по подключению библиотеки: https://jenkins.io/doc/book/pipeline/shared-libraries/#using-libraries

## Примеры Jenkinsfile

Если в настройках подключения shared-library включен флаг "Load implicitly":

```groovy
pipeline1C()
```

В обратном случае:

```groovy
@Library('jenkins-lib') _

pipeline1C()
```

> Да, вот и весь пайплайн. Конфигурирование через json.

## Внешний вид пайплайна в интерфейсе Blue Ocean

![image](https://user-images.githubusercontent.com/1132840/80956084-27b14400-8e09-11ea-9e92-683818a14503.png)

## Конфигурирование

По умолчанию применяется [файл конфигурации из ресурсов библиотеки](resources/globalConfiguration.json)

Поверх него накладывается конфигурация из файла `jobConfiguration.json` в корне проекта, если он присутствует.

Пример переопределения:

* указывается точная версия платформы (и соответственно метка агента, см. ограничения)
* идентификаторы credentials для пути к хранилищу и к паре логин/пароль для авторизации в хранилище (необходимы, если применяются шаги, работающие с информационной базой)
* включаются шаги запуска статического анализа SonarQube, валидации средствами EDT и синтаксического контроля 

```json
{
    "$schema": "https://raw.githubusercontent.com/firstBitSemenovskaya/jenkins-lib/master/resources/schema.json",
    "v8version": "8.3.14.1976",
    "secrets": {
        "storagePath": "f7b21c02-711a-4883-81c5-d429454e3f8b",
        "storage" : "c1fc5f33-67d4-493f-a2a4-97d3040e4b8c"
    },
    "stages": {
        "sonarqube": true,
        "edtValidation": true,
        "syntaxCheck": true
    }
}
```
