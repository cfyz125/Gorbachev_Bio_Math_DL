# Gorbachev_Bio_Math_DL
NN courses
\documentclass{article}
\usepackage[utf8]{inputenc}
\usepackage[T2A]{fontenc}
\usepackage[russian]{babel}
\usepackage{amssymb,amsmath}
\usepackage{graphicx}
\usepackage{epstopdf}
\usepackage{color}

\textwidth=170mm
\oddsidemargin=0mm
\textheight=240mm
\voffset=-25mm


\begin{document}
Горбачев Александр Workshop №1 тезисы.
Блок: Математика.
\section*{Предполагаемое название статьи}
$\qquad$Применение методов искусственного интеллекта для восстановления коэффициентов в обратной задаче для уравнения типа реакция-диффузия-адвекция с дополнительными данными в моделях самоорганизации в биологических системах.
\section*{Цели и задачи работы}

$\qquad$Биофизиками МГУ на основе работы \cite{BIO_Data} была построена следующая модель: 
\begin{equation}
	\label{MP}
	\left\{
	\begin{aligned}
		&\frac{\partial u}{\partial t} - \frac{\partial^2 u}{\partial x^2} = \lambda_1(u-v(x,t-t_0) - q(x))(u-\alpha(x)+\lambda_2 v(x,t-t_0)), \quad x \in (0,l), \quad t \in (0,T], \\
		&u(0,t) = 0, \quad u(1,t) = 0, \quad t \in (0,T], \\
		&u(x,0) = q(x), \quad x \in [0,l].
	\end{aligned}
	\right.
\end{equation}
Здесь, $\lambda_1, \lambda_2, t_0$ -- параметры, а функция $\alpha(x)$ является непрерывной при всех $x \in [0,l]$.
Функция $v(x,t)$ определяется следующим образом: она равна нулю при $t<0$, а при $t \geq 0 $ для каждого фиксированного $x \in [0,l]$, выполняющего роль параметра, определяется как решение задачи Коши:
\begin{equation*}
	\label{v_eq}
	\left\{
	\begin{aligned}
		&\frac{\partial v}{\partial t} = \varepsilon(-v(x,t) + \alpha(x)),  \quad t \in (0,T], \\
		&v(x,0) = 0.
	\end{aligned}
	\right.
\end{equation*}

Задача заключается в определении функции $0 \leq q(x) \leq \alpha(x)$ (из физических соображений), $ x \in [0,l]$, по известной дополнительной информации о значениях функции на известных кривых:
\begin{equation*}
	x_i(t) = f_1^i(t), \qquad u(x_i(t), t) = f_2^i(t),\qquad t\in [0, T],\qquad i = \overline{1, I}.
\end{equation*}

Ранее мной и нашей научной группой уже решались подобные задачи \cite{IPIL_10}, \cite{IPIL_11}. Основной идеей решения задачи была итерационная минимизация, основанная на градиентных методах для следующего функционала Тихонова:
\begin{equation*}
	J[q] = \sum_{i = 1}^{I}\int\limits_{0}^{T}\Big(u(f^i_1(t),t;q) - f_2^i(t)\Big)^2dt +\alpha\Omega[q].
\end{equation*}
 
А в качестве начального приближения для минимизации функционала выбиралось хорошее начальное приближение, выбранное с помощью асимптотических методов.
Для задачи в этой постановке возможность построения такого асимптотического приближения находится под вопросом, потому что дополнительной информацией ранее были функции "переходного"\  слоя $f_1(t)$. Однако в данной постановке такой информацией являются произвольные функции в области, на которой определена задача. В данной задаче может иметься несколько локальных минимумов, поэтому использование произвольного начального приближения для решения с помощью численных методов является неоптимальным.
Нами было выдвинуто предложение построить хорошее начальное приближение с помощью нейронной сети, которое будет уточнено градиентными методами. 
\section*{Описание данных}
$\qquad$ Для решения задачи вводится сетка $x_n = \dfrac{l}{N}n, t_m = \dfrac{T}{M}m, i = \overline{0, N}, j = \overline{0, M}$, где $N, M$ - количество узлов. Входными данными для нейронной сети будет $I$ пар функций ${f_1^i(t), f_2^i(t)}$, выходными данными будут значения заданной функции $q(x)$, в узлах сетки. Пример таких данных (\ref{fig:explanation_1}). Количество данных, для обучения может быть сгенерировано путем решения прямой задачи (\ref{MP}), для набора функций $q(x)$, которые будут задаваться следующим образом: 
\begin{equation*}
	\label{Data}
		q(x_n) =c + a_0(x_n - b) + a_1(x_n - b)^2 + a_2(x_n-b)^3 + a_3(x_n-b)^4 + a_4(x_n-b)^5 + a_5(x_n-b)^6, \qquad n=\overline{0,N} \\,
\end{equation*}
и если $q(x_n)$, меньше нуля, то данное значения ставится равным нулю, а если это значение больше $\alpha(x_i)$,то данный пример отбрасывается. Коэффициенты подбираются из некоторых промежутков, в согласовании с биофизиками, чтобы удовлетворять физическим свойствам задачи. Основное требование -- чтобы функция $q(x)$ выглядела, как одиночный пик, примерный вид зеленый график на (\ref{fig:explanation_2}).
Для валидационной и тестовой выборки планируется взять функции $q(x)$ в виде суммы пар функций Гаусса, как функции другого типа.
\begin{equation*}
	\label{Data}
	q(x_n) =d_0\exp\Big(-\dfrac{(x_n - e_0)^2}{f_0}\Big) + d_1\exp\Big(-\dfrac{(x_n-e_1)^2}{f_1}\Big), \qquad n=\overline{0,N} \\,
\end{equation*} 
Датасет будет состоять из 100000 функций для обучения, 20000 функций для валидации и 20000 для теста. Данные для обучения модели планируется зашумлять с некоторой относительной ошибкой.

Также остается вопрос выбора функций $f_1^i(t)$. Первый вариант -- обучить нейросеть на произвольных парах функций. Второй -- взять для задачи фиксированные экспериментальные функции $f_1^i(t)$, и затем обучать нейросеть на функциях $f_2^i(t)$. На данный момент выбран второй вариант, так как для его решения можно использовать более простую модель.
\section*{Метрики качества}
$\qquad$В качетве метрики будет использоваться $L_2$ (MSELoss), которая чаще всего используется для таких задач. Так как такая задача еще не была решена, то чем меньше мы получим ошибку, тем лучше. Использование нейронной сети позволит быстрее работать градиентному методу решения задачи, так как он сходится к правильному за бесконечное число итераций. Использование хорошего начального приближения позволит ускорить сходимость на приемлимую точность гораздо быстрее.
\section*{Модели и алгоритмы}
$\qquad$На данный момент уже есть несколько простых линейных и сверточных моделей, однако полученная с их помощью точность недостаточна. Линейная нейронная сеть была использована для похожей, но более простой обратной задачи \cite{Borz},  другая нейронная сеть применялась для определения коэффициентов при нахождении решения обратной задачи \cite{1}.   В представленной работе планируется использовать архитектуру ResNet \cite{ResNet}, с заменой двумерных сверток на одномерные.

\section*{Результаты}
$\qquad$На данный момент существует несколько простых линейных и сверточных нейронных сетей. Результат решения для сверточной нейронной сети представлен черным графиком на (\ref{fig:explanation_2}), и решение обратной задачи, полученное с их помощью и уточненное классическим алгоритмом, изображено на красном графике. В дальнейшем планируется улучшить результаты, используя более сложную структуру.
\begin{thebibliography}{9}
	\bibitem{BIO_Data}
	\textit{Uchimura A, Higuchi M, Minakuchi Y, Ohno M, Toyoda A, Fujiyama A, Miura I, Wakana S, Nishino J, Yagi T. Germline mutation rates and the long-term phenotypic effects of mutation accumulation in wild-type laboratory mice and mutator mice. Genome Res. 2015 Aug;25(8):1125-34. doi: 10.1101/gr.186148.114.}
	
	\bibitem{IPIL_10}
	\textit{Argun, R.L., Gorbachev, A.V., Lukyanenko, D.V. et al. Features of Numerical Reconstruction of a Boundary Condition in an Inverse Problem for a Reaction–Diffusion–Advection Equation with Data on the Position of a Reaction Front. Comput. Math. and Math. Phys. 62, 441–451 (2022). https://doi.org/10.1134/S0965542522030022}
	\bibitem{IPIL_11}
	\textit{Argun, R.; Gorbachev, A.; Levashova, N.; Lukyanenko, D. Inverse Problem for an Equation of the Reaction-Diffusion-Advection Type with Data on the Position of a Reaction Front: Features of the Solution in the Case of a Nonlinear Integral Equation in a Reduced Statement. Mathematics 2021, 9, 2342. https://doi.org/10.3390/math9182342}
	\bibitem{Borz}
	\textit{Lukyanenko, D.; Yeleskina, T.; Prigorniy, I.; Isaev, T.; Borzunov, A.; Shishlenin, M. Inverse Problem of Recovering the Initial Condition for a Nonlinear Equation of the Reaction–Diffusion–Advection Type by Data Given on the Position of a Reaction Front with a Time Delay. Mathematics 2021, 9, 342. https://doi.org/10.3390/math9040342}
	\bibitem{1}
	\textit{Gorbachenko, V.I., Lazovskaya, T.V., Tarkhov, D.A., Vasilyev, A.N., Zhukov, M.V. (2016). Neural Network Technique in Some Inverse Problems of Mathematical Physics. In: Cheng, L., Liu, Q., Ronzhin, A. (eds) Advances in Neural Networks – ISNN 2016. ISNN 2016. Lecture Notes in Computer Science(), vol 9719. Springer, Cham. https://doi.org/10.1007/978-3-319-40663-3\_36}
		\bibitem{ResNet}
	\textit{Sasha Targ, Diogo Almeida, Kevin Lyman, Resnet in Resnet: Generalizing Residual Architectures, CoRR, 2016, 
		https://doi.org/10.48550/arXiv.1603.08029}
\end{thebibliography}
%\newpage
\section*{Графики}
\begin{figure}[h]
	\caption{Примерный вид функций  $f^i_1(t)$, и соответсвующие им $f^i_2(t)$}
	\begin{minipage}[h]{0.49\linewidth}
		\center{\includegraphics[width=1\linewidth]{f_1.eps} \\ а)$f^i_1(t)$}
	\end{minipage}
	\hfill
	\begin{minipage}[h]{0.49\linewidth}
		\center{\includegraphics[width=1\linewidth]{f_2.eps} \\ б)$f^i_2(t)$}
	\end{minipage}
	\label{fig:explanation_1}
\end{figure}
\begin{figure}
	\caption{Результаты решения для модельного примера}
	\centering
	\includegraphics[width=0.60\textwidth]{Model_solv.pdf}\\
	\label{fig:explanation_2}
\end{figure}
%\begin{equation*}
%	\label{Data}
%	q(x) =d_0\exp\Big(-\dfrac{(x - e_0)^2}{f_0}\Big) + d_1\exp\Big(-\dfrac{(x-e_1)^2}{f_1}\Big), \qquad x\in[0,l] \\.
%\end{equation*} 
\end{document}
