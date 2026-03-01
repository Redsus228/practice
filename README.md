# Лабораторная работа №10: Тестирование приложения

## Описание задания

Необходимо написать комплекс тестов для проверки корректности работы ключевых компонентов приложения: репозитория, ViewModel и основных бизнес-процессов. Задание направлено на отработку навыков обеспечения качества кода через тестирование, что является критически важным для промышленной разработки Android-приложений.

## Исходные условия

- База: приложение из лабораторной работы №9 в ветке `task-9`
- Новое решение реализуется в ветке **`task-10`**
- Сохранить всю существующую функциональность приложения

## Основные функции

### Комплексное тестирование приложения

Написать тесты для следующих компонентов приложения:

#### 1. Тестирование репозитория (`DepositRepositoryTest`)
- Тест добавления нового расчёта
- Тест получения расчётов пользователя
- Тест получения истории расчётов
- Тест удаления расчёта
- Тест обновления существующего расчёта
- Тест валидации данных перед сохранением

#### 2. Тестирование ViewModel (`DepositViewModelTest`)
- Тест инициализации ViewModel
- Тест обновления состояния при добавлении расчёта
- Тест обработки ошибок ввода данных
- Тест фильтрации данных
- Тест сортировки расчётов (по дате, сумме)
- Тест обработки пустого списка расчётов

#### 3. Тестирование бизнес-логики расчёта вклада
- Тест корректности расчёта итоговой суммы
- Тест расчёта начисленных процентов
- Тест валидации входных параметров (стартовый взнос, срок, процентная ставка)
- Тест расчёта с ежемесячным пополнением
- Тест расчёта без пополнения

#### 4. Тестирование работы с авторизацией (`AuthViewModelTest`)
- Тест успешной авторизации
- Тест обработки ошибки авторизации
- Тест валидации полей ввода (логин, пароль)
- Тест сохранения токена
- Тест проверки состояния авторизации

## Технические требования

### 1. Unit-тесты (в папке `test`)

Реализовать тесты, которые проверяют логику классов без запуска Android-среды:

#### Пример unit-теста для репозитория:
```kotlin
@Test
fun `when deposit added then it appears in user deposits`() = runTest {
    // Given
    val userId = 1L
    val deposit = DepositCalculation(userId = userId, ...)
    
    // When
    repository.saveCalculation(deposit)
    val deposits = repository.getDepositsForUser(userId).first()
    
    // Then
    assertEquals(1, deposits.size)
    assertEquals(deposit.id, deposits[0].id)
}
```

#### Пример unit-теста для ViewModel:
```kotlin
@Test
fun `when valid deposit data entered then calculation is successful`() = runTest {
    // Given
    val viewModel = DepositViewModel(repository, calculationService)
    
    // When
    viewModel.setInitialAmount(10000.0)
    viewModel.setPeriodMonths(12)
    viewModel.setInterestRate(10.0)
    viewModel.calculate()
    
    // Then
    val state = viewModel.uiState.value
    assertTrue(state is DepositResultState.Success)
    assertEquals(11000.0, (state as DepositResultState.Success).finalAmount, 0.01)
}
```

#### Пример unit-теста для бизнес-логики:
```kotlin
@Test
fun `when calculate deposit with monthly top-up then final amount is correct`() {
    // Given
    val calculator = DepositCalculator()
    val initialAmount = 10000.0
    val monthlyTopUp = 1000.0
    val periodMonths = 12
    val interestRate = 12.0
    
    // When
    val result = calculator.calculate(
        initialAmount,
        monthlyTopUp,
        periodMonths,
        interestRate
    )
    
    // Then
    val expectedFinalAmount = 23143.0 // Расчёт по формуле
    assertEquals(expectedFinalAmount, result.finalAmount, 1.0)
}
```

### 2. Instrumented-тесты (в папке `androidTest`)

Реализовать тесты, которые выполняются на устройстве или эмуляторе и могут использовать Android-фреймворк:

#### Пример instrumented-теста:
```kotlin
@Test
fun `when user opens calculations screen then list is displayed`() {
    // Given
    val userId = 1L
    val testDeposits = listOf(
        DepositCalculation(userId = userId, ...),
        DepositCalculation(userId = userId, ...)
    )
    
    // Подготовка тестовых данных
    runBlocking {
        repository.saveCalculation(testDeposits[0])
        repository.saveCalculation(testDeposits[1])
    }
    
    // When
    launchFragmentInHiltContainer<MyCalculationsFragment> {
        MyCalculationsFragment(userId)
    }
    
    // Then
    onView(withId(R.id.calculations_list)).check(matches(isDisplayed()))
    onView(withId(R.id.calculations_list))
        .check(RecyclerViewItemCountAssertion(2))
}
```

## Реализационные требования

### 1. Покрытие кода
- Покрыть тестами **не менее 70%** ключевой логики
- Фокус на критические пути выполнения
- Проверка как успешных сценариев, так и обработки ошибок

### 2. Качество тестов
- Использовать **mock-объекты** для зависимостей (MockK или Mockito)
- Применять **корректные assertion'ы**
- Обеспечить **независимость тестов** (не зависят от порядка выполнения)
- Добавить **описательные имена** тестам
- Следовать паттерну **Given-When-Then**
- Проверять **граничные значения** и **негативные сценарии**

### 3. Организация тестов
- Разместить unit-тесты в `src/test/java`
- Разместить instrumented-тесты в `src/androidTest/java`
- Группировать тесты по компонентам
- Использовать `@Before` и `@After` для настройки и очистки
- При необходимости использовать `@BeforeClass` и `@AfterClass`

## Дополнительно

- Добавить тесты для экрана авторизации через QR-код
- Реализовать UI-тесты с помощью Espresso для ключевых сценариев
- Добавить снимки интерфейса (snapshot testing)
- Настроить генерацию отчёта о покрытии кода (JaCoCo)
- Реализовать параметризованные тесты для проверки различных входных данных
- Добавить тесты для работы с локализацией

---

*Примечание: данное задание направлено на отработку навыков обеспечения качества кода через тестирование, что является критически важным для промышленной разработки Android-приложений. Тестирование помогает выявлять ошибки на ранних этапах, обеспечивает стабильность приложения при рефакторинге и упрощает поддержку кода в долгосрочной перспективе.*