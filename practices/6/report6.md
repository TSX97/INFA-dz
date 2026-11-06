# Практическая работа №6

## Программирование. Анализ результатов измерений

**Цель работы:** освоение базовых конструкций языка программирования Go для обработки экспериментальных данных.

## 1. Постановка задачи

Разработать программу для статистической обработки результатов физических измерений.

Программа должна выполнять следующие действия:

1. Запрашивать количество измерений от 5 до 20.
2. Считывать результаты измерений.
3. Вычислять среднее арифметическое.
4. Находить минимальное и максимальное значения.
5. Вычислять размах измерений.
6. Вычислять среднеквадратическое отклонение.
7. Определять относительную погрешность в процентах.
8. Выполнять фильтрацию выбросов.

## 2. Используемые средства

- Язык программирования: Go.
- Пакеты: `fmt`, `bufio`, `os`, `math`.
- Слайсы для хранения результатов измерений.
- Функции для выполнения вычислений.
- Циклы и условные операторы для обработки данных.

## 3. Исходный код программы

```go
package main

import (
	"bufio"
	"fmt"
	"math"
	"os"
)

const MaxMeasurements = 20

type Statistics struct {
	Average       float64
	Min           float64
	Max           float64
	Range         float64
	StdDev        float64
	RelativeError float64
}

func calculateStatistics(data []float64) Statistics {
	var stats Statistics

	stats.Min = data[0]
	stats.Max = data[0]

	for _, value := range data {
		stats.Average += value

		if value < stats.Min {
			stats.Min = value
		}
		if value > stats.Max {
			stats.Max = value
		}
	}

	stats.Average /= float64(len(data))
	stats.Range = stats.Max - stats.Min

	var sumSquares float64

	for _, value := range data {
		diff := value - stats.Average
		sumSquares += diff * diff
	}

	stats.StdDev = math.Sqrt(sumSquares / float64(len(data)))

	if stats.Average != 0 {
		stats.RelativeError = stats.StdDev /
			math.Abs(stats.Average) * 100
	}

	return stats
}

func filterOutliers(data []float64, stats Statistics) []float64 {
	var filtered []float64
	limit := 2 * stats.StdDev

	for _, value := range data {
		if math.Abs(value-stats.Average) <= limit {
			filtered = append(filtered, value)
		}
	}

	return filtered
}

func main() {
	reader := bufio.NewReader(os.Stdin)

	fmt.Println("=== АНАЛИЗ РЕЗУЛЬТАТОВ ИЗМЕРЕНИЙ ===")
	fmt.Print("Введите количество замеров (5-20): ")

	var n int

	if _, err := fmt.Fscan(reader, &n); err != nil ||
		n < 5 || n > MaxMeasurements {
		fmt.Println("Ошибка: количество замеров должно быть от 5 до 20")
		return
	}

	measurements := make([]float64, n)

	fmt.Println("Введите результаты измерений:")

	for i := 0; i < n; i++ {
		fmt.Printf("Измерение %d: ", i+1)

		if _, err := fmt.Fscan(reader, &measurements[i]); err != nil ||
			math.IsNaN(measurements[i]) ||
			math.IsInf(measurements[i], 0) {
			fmt.Println("Ошибка: некорректное значение")
			return
		}
	}

	stats := calculateStatistics(measurements)

	fmt.Println("\n=== РЕЗУЛЬТАТЫ АНАЛИЗА ===")
	fmt.Printf("Количество измерений: %d\n", n)
	fmt.Printf("Среднее значение: %.3f\n", stats.Average)
	fmt.Printf("Минимальное значение: %.3f\n", stats.Min)
	fmt.Printf("Максимальное значение: %.3f\n", stats.Max)
	fmt.Printf("Размах: %.3f\n", stats.Range)
	fmt.Printf("Среднеквадратическое отклонение: %.3f\n", stats.StdDev)

	if stats.Average != 0 {
		fmt.Printf("Относительная погрешность: %.1f%%\n",
			stats.RelativeError)
	} else {
		fmt.Println("Относительная погрешность: не определена")
	}

	filtered := filterOutliers(measurements, stats)

	fmt.Println("\n=== ФИЛЬТРАЦИЯ ВЫБРОСОВ ===")
	fmt.Printf("Допустимый диапазон: %.3f ± %.3f\n",
		stats.Average, 2*stats.StdDev)

	fmt.Println("Отфильтрованные данные:")

	for _, value := range filtered {
		fmt.Printf("%.3f ", value)
	}

	fmt.Printf("\nОсталось измерений после фильтрации: %d\n",
		len(filtered))
}
```
