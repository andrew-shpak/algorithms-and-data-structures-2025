# Читання файлу
```csv 
ім'я
"Іван Петро"
"Олена Шевченко"
"Петро Іваненко"
```
## Відкритання файлу
``` С++ 
 	ifstream file(path);
    if (!file.is_open()) {
        cerr << "Could not open file " << path << endl;
        return false;
    }
```
## Читання файлу 
``` С++ 
	vector<string> user_names(3);
	....
	string line;
    getline(file, line); // пропускаємо перший рядок
 	while (getline(file, line)) {
        constexpr char SEPARATOR = ',';
        if (line.empty()) {
            continue;

        stringstream row(line); // читаємо весь рядок
		string name; // створюємо тимчасову змінну 

 		std::getline(row, name, SEPARATOR); // читаємо перший елемент та записуємо у тимчасову зміну

		... // Додаткова логіка

		 user_names.push_back(name);
 	}	
```
## Прохід по колонці яка містить vector значень
``` С++ 
	stringstream ss(column_value);
    while (ss >> value) {
        // logic
    }
```
## Користні методи string
``` С++ 
	sting = "algoritms"
	- text.front() - повертає перший елемент (Поверне - 'a')
	- text.back() - повертає останній елемент (Поверне - 's')
	- stoi - перетворення стрічки у int
	- stod - перетворює стрічку у double
```
## Ініціалізація vector
``` С++ 
	std::vector<int> v1;        // порожній вектор
	std::vector<int> v2{};      //  порожній  
	std::vector<int> v3 = {};   // теж порожній
	std::vector<int> v1(5);     // 5 елементів, всі = 0
	std::vector<int> v1{1, 2, 3, 4, 5};     //ініціалізація
	std::vector<int> v2 = {1, 2, 3, 4, 5};  // ініціалізація
	std::vector<int> v2{10, 20, 30};
	std::vector<int> v3(v2.begin(), v2.end()); // ініціалізація на основі іншого вектору
```