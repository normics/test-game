# test-game
тестовая игра
using UnityEngine;
using TMPro; // Для работы с TextMeshPro

public class ClickerManager : MonoBehaviour
{
    // ... (Остальные переменные из предыдущего шага остаются без изменений) ...
    // Очки
    public int score = 0;
    public TextMeshProUGUI scoreText;
    public int scoreToWin = 10000; // Цель для победного сообщения

    // Параметры для улучшения клика
    public int clickPower = 1;
    public int clickUpgradeCost = 10;
    public TextMeshProUGUI clickUpgradeCostText;
    public int clickUpgradesCount = 0; // Новая переменная для статистики клик-прокачек

    // Параметры для пассивного дохода
    public int passiveIncome = 0;
    public int passiveUpgradeCost = 20;
    public TextMeshProUGUI passiveUpgradeCostText;
    public float passiveInterval = 1f;
    private float timer = 0f;
    public int passiveUpgradesCount = 0; // Новая переменная для статистики пассивных прокачек

    // Статистика
    public int totalClicks = 0; // Новая переменная для общего числа кликов
    public TextMeshProUGUI totalClicksText;
    public TextMeshProUGUI clickUpgradesCountText;
    public TextMeshProUGUI passiveUpgradesCountText;

    // Панель победного сообщения
    public GameObject winPanel; // Ссылка на наш GameObject WinPanel

    // Флаг, чтобы сообщение появилось только один раз
    private bool hasWon = false;

    // --- Переменные для меню разработчика ---
    public GameObject developerAccessPanel; // Панель для ввода кода
    public TMP_InputField codeInputField; // Поле для ввода кода
    public int developerCode = 90; // Наш секретный код (0090, но int хранит 90)

    public GameObject developerMenuPanel; // Панель с читами

    void Start()
    {
        LoadGame(); // Загружаем данные при старте
        UpdateUI(); // Обновляем весь UI
        winPanel.SetActive(false); // Убедимся, что панель победы скрыта при старте
        developerAccessPanel.SetActive(false); // Скрываем панель доступа при старте
        developerMenuPanel.SetActive(false); // Скрываем меню разработчика при старте
    }

    void Update()
    {
        // Логика для пассивного дохода
        timer += Time.deltaTime;
        if (timer >= passiveInterval)
        {
            score += passiveIncome;
            UpdateUI();
            timer = 0f;
            Debug.Log("Пассивный доход! Текущие очки: " + score);
            SaveGame(); // Сохраняем игру после пассивного дохода
        }

        // Проверяем условие победы
        if (score >= scoreToWin && !hasWon)
        {
            ShowWinMessage();
        }

        // Проверяем нажатие горячей клавиши для меню разработчика (например, 'D')
        if (Input.GetKeyDown(KeyCode.D)) // Можно выбрать любую другую удобную клавишу
        {
            ToggleDeveloperAccessPanel();
        }
    }

    public void OnClickButton()
    {
        totalClicks++; // Увеличиваем счетчик кликов
        score += clickPower;
        UpdateUI();
        Debug.Log("Клик! Текущие очки: " + score);
        SaveGame();
    }

    public void BuyClickUpgrade()
    {
        if (score >= clickUpgradeCost)
        {
            score -= clickUpgradeCost;
            clickPower++;
            clickUpgradeCost = Mathf.RoundToInt(clickUpgradeCost * 1.5f);
            clickUpgradesCount++; // Увеличиваем счетчик прокачек клика
            UpdateUI();
            SaveGame();
            Debug.Log("Улучшение клика куплено! Новая сила клика: " + clickPower);
        }
        else
        {
            Debug.Log("Недостаточно очков для улучшения клика!");
        }
    }

    public void BuyPassiveUpgrade()
    {
        if (score >= passiveUpgradeCost)
        {
            score -= passiveUpgradeCost;
            passiveIncome++;
            passiveUpgradeCost = Mathf.RoundToInt(passiveUpgradeCost * 1.8f);
            passiveUpgradesCount++; // Увеличиваем счетчик пассивных прокачек
            UpdateUI();
            SaveGame();
            Debug.Log("Улучшение пассивного дохода куплено! Новый пассив: " + passiveIncome);
        }
        else
        {
            Debug.Log("Недостаточно очков для улучшения пассивного дохода!");
        }
    }

    // Обновляем все тексты UI, включая статистику
    void UpdateUI()
    {
        scoreText.text = "Очки: " + score;
        clickUpgradeCostText.text = "Цена: " + clickUpgradeCost;
        passiveUpgradeCostText.text = "Цена: " + passiveUpgradeCost;

        // Обновляем статистику
        totalClicksText.text = "Всего кликов: " + totalClicks;
        clickUpgradesCountText.text = "Прокачек клика: " + clickUpgradesCount;
        passiveUpgradesCountText.text = "Прокачек пассива: " + passiveUpgradesCount;
    }

    // Показываем победное сообщение
    void ShowWinMessage()
    {
        hasWon = true; // Устанавливаем флаг, чтобы не спамить
        winPanel.SetActive(true); // Активируем панель
        // Можно тут добавить остановку игры или что-то ещё, если нужно
    }

    // --- Методы для сохранения и загрузки (обновленные) ---
    void SaveGame()
    {
        PlayerPrefs.SetInt("Score", score);
        PlayerPrefs.SetInt("ClickPower", clickPower);
        PlayerPrefs.SetInt("ClickUpgradeCost", clickUpgradeCost);
        PlayerPrefs.SetInt("PassiveIncome", passiveIncome);
        PlayerPrefs.SetInt("PassiveUpgradeCost", passiveUpgradeCost);

        // Сохраняем статистику
        PlayerPrefs.SetInt("TotalClicks", totalClicks);
        PlayerPrefs.SetInt("ClickUpgradesCount", clickUpgradesCount);
        PlayerPrefs.SetInt("PassiveUpgradesCount", passiveUpgradesCount);
        PlayerPrefs.SetInt("HasWon", hasWon ? 1 : 0); // Сохраняем состояние флага победы

        PlayerPrefs.Save();
        Debug.Log("Игра сохранена!");
    }

    void LoadGame()
    {
        if (PlayerPrefs.HasKey("Score"))
        {
            score = PlayerPrefs.GetInt("Score");
            clickPower = PlayerPrefs.GetInt("ClickPower");
            clickUpgradeCost = PlayerPrefs.GetInt("ClickUpgradeCost");
            passiveIncome = PlayerPrefs.GetInt("PassiveIncome");
            passiveUpgradeCost = PlayerPrefs.GetInt("PassiveUpgradeCost");

            // Загружаем статистику
            totalClicks = PlayerPrefs.GetInt("TotalClicks");
            clickUpgradesCount = PlayerPrefs.GetInt("ClickUpgradesCount");
            passiveUpgradesCount = PlayerPrefs.GetInt("PassiveUpgradesCount");
            hasWon = PlayerPrefs.GetInt("HasWon") == 1; // Загружаем состояние флага победы

            Debug.Log("Игра загружена!");
        }
        else
        {
            Debug.Log("Сохраненных данных нет, начинаем новую игру.");
        }
    }

    // --- Метод для сброса всего ---
    public void ResetGame()
    {
        PlayerPrefs.DeleteAll(); // Удаляет все сохраненные PlayerPrefs данные
        Debug.Log("Все данные игры сброшены!");

        // Сбрасываем все переменные к начальным значениям
        score = 0;
        clickPower = 1;
        clickUpgradeCost = 10;
        passiveIncome = 0;
        passiveUpgradeCost = 20;
        totalClicks = 0;
        clickUpgradesCount = 0;
        passiveUpgradesCount = 0;
        hasWon = false;

        UpdateUI(); // Обновляем UI, чтобы показать сброшенные значения
        winPanel.SetActive(false); // Скрываем панель победы, если она была активна
        developerAccessPanel.SetActive(false); // Скрываем панель доступа
        developerMenuPanel.SetActive(false); // Скрываем меню разработчика
    }

    // --- Методы для меню разработчика ---

    // Переключает видимость панели ввода кода
    public void ToggleDeveloperAccessPanel()
    {
        // Если панель меню разработчика уже открыта, закрываем ее.
        // Если панель доступа уже открыта, закрываем ее.
        // Иначе открываем панель доступа.
        if (developerMenuPanel.activeSelf)
        {
            developerMenuPanel.SetActive(false);
            developerAccessPanel.SetActive(false);
        }
        else if (developerAccessPanel.activeSelf)
        {
            developerAccessPanel.SetActive(false);
        }
        else
        {
            developerAccessPanel.SetActive(true);
            codeInputField.text = ""; // Очищаем поле ввода
        }
    }

    // Проверяет введенный код
    public void CheckDeveloperCode()
    {
        int enteredCode;
        // Пытаемся преобразовать введенный текст в число
        if (int.TryParse(codeInputField.text, out enteredCode))
        {
            if (enteredCode == developerCode)
            {
                Debug.Log("Код разработчика верен! Открываем меню.");
                developerAccessPanel.SetActive(false); // Скрываем панель доступа
                developerMenuPanel.SetActive(true);    // Показываем меню разработчика
            }
            else
            {
                Debug.Log("Неверный код разработчика.");
                codeInputField.text = ""; // Очищаем поле для повторного ввода
            }
        }
        else
        {
            Debug.Log("Введите только цифры!");
            codeInputField.text = ""; // Очищаем поле
        }
    }

    // Выдача очков
    public void AddScoreCheat()
    {
        score += 1000; // Добавляем 1000 очков
        UpdateUI();
        SaveGame();
        Debug.Log("Добавлено 1000 очков читом. Текущие очки: " + score);
    }

    // Прокачка клика читом
    public void UpgradeClickCheat()
    {
        clickPower += 10; // Увеличиваем силу клика на 10
        UpdateUI();
        SaveGame();
        Debug.Log("Клик прокачан читом. Новая сила клика: " + clickPower);
    }

    // Прокачка пассива читом
    public void UpgradePassiveCheat()
    {
        passiveIncome += 5; // Увеличиваем пассивный доход на 5
        UpdateUI();
        SaveGame();
        Debug.Log("Пассив прокачан читом. Новый пассив: " + passiveIncome);
    }

    // Закрыть меню разработчика
    public void CloseDeveloperMenu()
    {
        developerMenuPanel.SetActive(false); // Скрываем меню
    }
}
