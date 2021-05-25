---
title: C++ std::as_const
---

```C++
#include <iostream>
#include <vector>
using namespace std;
class SpreadsheetCell
{
private:
    double mValue;
public:
    SpreadsheetCell(double d) : mValue(d) {}
    void print() const { cout << mValue << endl; }
};

class Spreadsheet
{
private:
    vector<vector<SpreadsheetCell>> mCells;
public:
    Spreadsheet(size_t x, size_t y) : mCells(x, vector<SpreadsheetCell>(y, 0)) {}
    SpreadsheetCell& getCellAt(size_t x, size_t y);
    const SpreadsheetCell& getCellAt(size_t x, size_t y) const;

    void setCellAt(size_t x, size_t y, double d) { verifyCoordinate(x, y); mCells[x][y] = d; }

    void verifyCoordinate(size_t x, size_t y) const;
};
void Spreadsheet::verifyCoordinate(size_t x, size_t y) const
{
    if(x >= mCells.size()) throw std::out_of_range("");
    if(y >= mCells[x].size()) throw std::out_of_range("");
}
const SpreadsheetCell& Spreadsheet::getCellAt(size_t x, size_t y) const
{
    verifyCoordinate(x, y);
    return mCells[x][y];
}
SpreadsheetCell& Spreadsheet::getCellAt(size_t x, size_t y)
{
    return const_cast<SpreadsheetCell&>(std::as_const(*this).getCellAt(x, y));
}
int main()
{
    Spreadsheet sheet1(5, 6);
    sheet1.setCellAt(1, 1, 100);
    SpreadsheetCell& cell1 = sheet1.getCellAt(1, 1);
    cell1.print();

    const Spreadsheet sheet2(5, 6);
    const SpreadsheetCell& cell2 = sheet2.getCellAt(1, 1);
    cell2.print();
}
```
