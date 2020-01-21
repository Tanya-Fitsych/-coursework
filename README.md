# -coursework
 coursework
//---------------------------------------------------------------------------
#include <vcl.h>
#include <math.h>
#pragma hdrstop
#include "Unit2.h"
//---------------------------------------------------------------------------
#pragma package(smart_init)
#pragma resource "*.dfm"
TFormMain *FormMain;
//---------------------------------------------------------------------------
// Конструктор форми
__fastcall TFormMain::TFormMain(TComponent* Owner): TForm(Owner)
{
	// Заповнення списка еталонних образів.
	for (int i = 0; i < 10; i++)
	{
		ComboBoxSamples->Items->Append(i);
	}
// Формування підписів таблиць.
	StringGrid1->Cells[0][0] = "Нейр\\Вага";
	StringGrid2->Cells[0][0] = "Нейр\\Вага";
	StringGrid3->Cells[0][0] = "Нейр\\Вага";
	for (int i = 1; i < 51; i++)
	{
		StringGrid1->Cells[i][0] = i;
		StringGrid1->Cells[0][i] = i;
		StringGrid2->Cells[i][0] = i;
		StringGrid2->Cells[0][i] = i;
		StringGrid3->Cells[i][0] = i;
	}
	for (int i = 0; i < 10; i++)
	{
		StringGrid3->Cells[0][i + 1] = i + 1;
		StringGridOut->Cells[0][i] = i;
	}
}
//---------------------------------------------------------------------------
// Вибір прикладу образа зі списку.
void __fastcall TFormMain::ComboBoxSamplesChange(TObject *Sender)
{
// Подача вибраного прикладу із навчальної вибірки на вхід нейромережі.
	for (int i = 0; i < 10; i++)
	{
		for (int j = 0; j < 5; j++)
		{
			input[i][j] = LearningSamples[ComboBoxSamples->Text.ToInt()][i][j] == 1 ? HI : LO;
		}
	}

	// Вивід зображення.
	DrawGridPattern->Refresh();
	// Вивід результату.
	LabelOut->Caption = "";
}
//---------------------------------------------------------------------------
	   // Замальовування точки вхідного образу.
void __fastcall TFormMain::DrawGridPatternDrawCell(TObject *Sender, int ACol, int ARow,TRect &Rect, TGridDrawState State)
{
  // Установка коліру точки вхідного образу.
  DrawGridPattern->Canvas->Brush->Color = input[ARow][ACol] == HI ? clBlue : clRed;
  DrawGridPattern->Canvas->FillRect(Rect);
}
		   // Вибір точки вхідного образа.
void __fastcall TFormMain::DrawGridPatternSelectCell(TObject *Sender, int ACol, int ARow, bool &CanSelect)
{
	//Інвертування стану точки вхідного образу.
	input[ARow][ACol] = input[ARow][ACol] == HI ? LO : HI;
	// вивід рузультату.
	LabelOut->Caption = "";
//---------------------------------------------------------------------------
// Натискання кнопки "Задати коефіцієнти".
void __fastcall TFormMain::ButtonSetWeightsClick(TObject *Sender)
{
	//Генерація випадкових вагових коефіцієнтів нейронів.
	RandomizeWeights();
	//Вивід значень вагових коефіцієнтів нейронів.
	PrintWeights();
	// Вивід результата.
	LabelOut->Caption = "";
	//
	ButtonTrainNeuronet->Enabled = true;
}
// Натискання кнопки "Навчити нейромережу"
void __fastcall TFormMain::ButtonTrainNeuronetClick(TObject *Sender)
{
	//  Навчання шарів нейромережі сигнальним методом Хебба.
	SignalHebbianLearning();
	//Вивід значень вагових коефіцієнтів нейронів шарів
	PrintWeights();
	// Вивід результата.
	LabelOut->Caption = "";
	ButtonTrainPerceptron->Enabled = true;
}

}
	
		//Натискання кнопки "навчити персептрон"
	void __fastcall TFormMain::ButtonTrainPerceptronClick(TObject *Sender)
{
	// Навчання персептрона.
	PerceptronBasedLearning();
	// Вивід значень вагових коефіцієнтів нейронів шарів.
	PrintWeights();
	// Вивід результата.
	LabelOut->Caption = "";
	ButtonRecognize->Enabled = true;
}
	 
	 // Натискання кнопки "Розпізнати  образ".
void __fastcall TFormMain::ButtonRecognizeClick(TObject *Sender)
{
// Результат распознавания образа.
	int result;
	// Максимальне значення виходу персептрона (ініціалізується мінімальним).
	double max = -1;
	// Розповсюдження сигналів по шарам нейромережі.
	Propagate();
	// Классифікація персептроном вихідних сигналів посліднього шару.
	Classificate();
	// Вивід значень виходів персептрона.
	PrintOut();
	// Пошук серед виходів персептрона елемента з максимальним значенням.
	for (int i = 0; i < 10; i++)
	{
		if (max < outP[i])
		{
			max = outP[i];
			result = i;
		}
	}
	// Вивід результата.
	LabelOut->Caption = result;
}

//---------------------------------------------------------------------------
				// Вивід значень вагових коефіцієнтів нейронів.
void __fastcall TFormMain::PrintWeights()
{
	// Вивід значень вагових коефіцієнтів нейронів шарів.
	for (int i = 0; i < 50; i++)
	{
		for (int j = 0; j < 50; j++)
		{
			StringGrid1->Cells[j + 1][i + 1] = weight[0][i][j];
		}
	}
	for (int i = 0; i < 50; i++)
	{
		for (int j = 0; j < 50; j++)
		{
			StringGrid2->Cells[j + 1][i + 1] = weight[1][i][j];
		}
	}
	// Вивід значень вагових коефіцієнтів нейронів персептрона.
	for (int i = 0; i < 10; i++)
	{
		for (int j = 0; j < 50; j++)
		{
			StringGrid3->Cells[j + 1][i + 1] = weightP[i][j];
		}
	}
}
//---------------------------------------------------------------------------
// Вивід значень виходів персептрона.
void __fastcall TFormMain::PrintOut()
{
	// Вивід виходів нейромережі.
	for (int i = 0; i < 10; i++)
	{
		StringGridOut->Cells[1][i] = outP[i];
	}
}
//---------------------------------------------------------------------------
//Генерація випадкових вагових коефіцієнтів нейронів.
void __fastcall TFormMain::RandomizeWeights()
{
	// Верхня границя діапазона.
	double boundUpper = 1.0;
	// Нижня границя диапазона.
	double boundLower = -1.0;

	// Генерація псевдовипадкових вагових коефіцієнтів нейронних шарів.
	for (int n = 0; n < 2; n++)
	{
		for (int i = 0; i < 50; i++)
		{
			for (int j = 0; j < 50; j++)
			{
				weight[n][i][j] = (double)rand() / (RAND_MAX) * (boundUpper - boundLower) + boundLower;
			}
		}
	}

	// Генерациія псевдовипадкових вагових коефіцієнтів нейронів персептрона.
	for (int i = 0; i < 10; i++)
	{
		for (int j = 0; j < 50; j++)
		{
			weightP[i][j] = (double)rand() / (RAND_MAX) * (boundUpper - boundLower) + boundLower;
		}
	}
}
//---------------------------------------------------------------------------
// Розповсюдження сигналів по шарах нейромережі.
void __fastcall TFormMain::Propagate()
{
	// Перебір нейронів 1-го шару.
	for (int i = 0; i < 50; i++)
	{
		// Зважена сума входів нейрона.
		double sum = 0;

		// Перебір входів і ваги нейрона.
		for (int j = 0; j < 50; j++)
		{
			// Обчислення зваженої суми входів нейрона.
			sum += input[j / 5][j % 5] * weight[0][i][j];
		}

		// Напівлінійна активаційна функція.
		if (sum > 0)
		{
			out[0][i] = kappa * sum;
		}
		else
		{
			out[0][i] = 0;
		}
	}
	// Перебір нейронів 2-го шару.
	for (int i = 0; i < 50; i++)
	{
		// Зважена сума входів нейрона.
		double sum = 0;

		// Перебір входів і ваги нейрона.
		for (int j = 0; j < 50; j++)
		{
			// Обчислення зваженої суми входів нейрона.
			sum += out[0][j] * weight[1][i][j];
		}

	// Напівлінійна активаційна функція.
		if (sum > 0)
		{
			out[1][i] = kappa * sum;
		}
		else
		{
			out[1][i] = 0;
		}
	}
}
//---------------------------------------------------------------------------
// Класифікація вихідних сигналів нейромережі.
void __fastcall TFormMain::Classificate()
{
	// Перебір нейронів шару.
	for (int i = 0; i < 10; i++)
	{
		// Зважена сума входів нейрона.
		double sum = 0;

		// Перебір входів і ваги нейрона.
		for (int j = 0; j < 50; j++)
		{
	// Обчислення зваженої суми входів нейрона.
			sum += out[1][j] * weightP[i][j];
		}
		// Логічна активаційна функція.
		outP[i] = 1 / (1 + exp(-2 * sum));
	}
}
//---------------------------------------------------------------------------

//---------------------------------------------------------------------------
//Навчання за алгоритмом навчання одношарового персептрона.
void __fastcall TFormMain::PerceptronBasedLearning()
{
	//Ознака помилки персептрона.
	bool error;

	// Цикл навчання персептрона всім образам з навчальної вибірки (приблизно 100 раз для кожного способу).
	for (int s = 0; s < 1000; s++)
	{
		// Номер образу з навчальної вибірки.
		int num = random(10);
		//Подача образу на вхід нейромережі.
		for (int i = 0; i < 10; i++)
		{
			for (int j = 0; j < 5; j++)
			{
				input[i][j] = LearningSamples[num][i][j] == 1 ? HI : LO;
			}
		}

		// Поширення сигналів по нейромережі.
		Propagate();
		// Цикл навчання персептрона заданому образу з навчальної вибірки, поки персептрон помиляється.
		do
		{
			// Класифікація вихідних сигналів нейромережі.
			Classificate();
			// Скидання ознаки помилки.
			error = false;
			// Перевірка виходів персептрона.
			for (int i = 0; i < 10; i++)
			{
				if (num == i)
				{
					if (outP[i] < 0.7)
						error = true;
				}
				else
				{
					if (outP[i] > 0.3)
						error = true;
				}
			}
			if (error)
			{
			// Перевірка виходів персептрона.
			for (int i = 0; i < 10; i++)
			{
				// Різниця між необхідним і отриманим значенням виходу нейрона.
				double sigma = (num == i ? 1.0 : -0.0) - outP[i];
				// Навчання персептрона, якщо різниця не нульова.
				if (sigma)
				{
					//Установка ознаки помилки.
					error = true;
					//Перебір входів і ваг нейрона.
					for (int j = 0; j < 50; j++)
					{
						// Корекція вагових коефіцієнтів нейронів персептрона.
						weightP[i][j] += eta * sigma * out[1][j];
					}
				}
			}
			}
		}
		while (error);
	}
