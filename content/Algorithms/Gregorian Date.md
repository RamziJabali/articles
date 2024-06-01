---
title: Gregorian Date
date: 2019-10-19
tags:
  - algorithm
  - java
  - medium
---
```java
public class DateUtil {
    private boolean isItALeapYear = false;
    private int holidaysWorked;
    private int workDaysWorked;

    public boolean checkGregorianDate(String inDate) {
        String day, month, year;
        int _day, _month, _year;
        if (inDate.length() == 8) {
            day = inDate.charAt(6) + "" + inDate.charAt(7);
            month = inDate.charAt(4) + "" + inDate.charAt(5);
            year = inDate.charAt(0) + "" + inDate.charAt(1) + "" + inDate.charAt(2) + "" + inDate.charAt(3);
            _day = getValueOrNegativeOneForIntegers(day);
            _month = getValueOrNegativeOneForIntegers(month);
            _year = getValueOrNegativeOneForIntegers(year);
        } else {
            println("Enter a correct date EX(yyyy/mm/yy --> 03 07 2000) 8 CHARACTERS");
            return false;
        }

        if (_day < 1 || _month < 1 || _month > 12 || _year < 1) {
            println("Enter an integer that works please, Thank you!");
            return false;
        }

        if (!doesDayFitInMonth(_day, _month, _year)) {
            return false;
        }
        println("Your date is good!");
        return true;
    }

    public int getWorkDaysWorked() {
        return workDaysWorked;
    }

    public int getHolidaysWorked() {
        return holidaysWorked;
    }

    public void daysTypeCount(String startWork, String endWork, String startHoliday, String endHoliday) {
        int startDay = getValueOrNegativeOneForIntegers(startWork.charAt(6) + "" + startWork.charAt(7));
        int endDay = getValueOrNegativeOneForIntegers(endWork.charAt(6) + "" + endWork.charAt(7));
        int startMonth = getValueOrNegativeOneForIntegers(startWork.charAt(4) + "" + startWork.charAt(5));
        int endMonth = getValueOrNegativeOneForIntegers(endWork.charAt(4) + "" + endWork.charAt(5));
        int startYear = getValueOrNegativeOneForIntegers(startWork.charAt(0) + "" + startWork.charAt(1) + "" + startWork.charAt(2) + "" + startWork.charAt(3));
        int endYear = getValueOrNegativeOneForIntegers(endWork.charAt(0) + "" + endWork.charAt(1) + "" + endWork.charAt(2) + "" + endWork.charAt(3));
        int startDayHoliday = getValueOrNegativeOneForIntegers(startHoliday.charAt(6) + "" + startHoliday.charAt(7));
        int endDayHoliday = getValueOrNegativeOneForIntegers(endHoliday.charAt(6) + "" + endHoliday.charAt(7));
        int startMonthHoliday = getValueOrNegativeOneForIntegers(startHoliday.charAt(4) + "" + startHoliday.charAt(5));
        int endMonthHoliday = getValueOrNegativeOneForIntegers(endHoliday.charAt(4) + "" + endHoliday.charAt(5));
        int startYearHoliday = getValueOrNegativeOneForIntegers(startHoliday.charAt(0) + "" + startHoliday.charAt(1) + "" + startHoliday.charAt(2) + "" + startHoliday.charAt(3));
        int endYearHoliday = getValueOrNegativeOneForIntegers(endHoliday.charAt(0) + "" + endHoliday.charAt(1) + "" + endHoliday.charAt(2) + "" + endHoliday.charAt(3));

        //Todo: add if one of the dates are equal and the other is not
        if (endYearHoliday < startYear || startYearHoliday < endYear) {//days worked aren't aligned with holidays
            setHolidaysWorked("0");
            setWorkDaysWorked(daysDiff(startWork, endWork));
        } else if (startYear > startYearHoliday && endYear < endYearHoliday) {//he worked through out the holiday
            setHolidaysWorked(daysDiff(startWork, endWork));
            setWorkDaysWorked("0");//days worked on normal work days == 0;
        } else if (startYear < startYearHoliday && endYear > endYearHoliday) {
            setHolidaysWorked(daysDiff(startHoliday, endHoliday));
            workDaysWorked = Math.abs(holidaysWorked - getValueOrNegativeOneForIntegers(daysDiff(startWork, endHoliday)));
        } else if (startYear > startYearHoliday && endYear > endYearHoliday) {
            holidaysWorked = getValueOrNegativeOneForIntegers(daysDiff(startWork, endHoliday));
            workDaysWorked = getValueOrNegativeOneForIntegers(daysDiff(endHoliday, endWork));
        } else if (startYear < startYearHoliday && endYear < endYearHoliday) {
            holidaysWorked = getValueOrNegativeOneForIntegers(daysDiff(startHoliday, endWork));
            workDaysWorked = getValueOrNegativeOneForIntegers(daysDiff(startWork, startHoliday));
        } else if (startYear == startYearHoliday && endYear < endYearHoliday) {
            holidaysWorked = getValueOrNegativeOneForIntegers(daysDiff(startWork, endWork));
            workDaysWorked = 0;
        } else if (startYear == startYearHoliday && endYear > endYearHoliday) {
            holidaysWorked = getValueOrNegativeOneForIntegers(daysDiff(startWork, endHoliday));
            workDaysWorked = getValueOrNegativeOneForIntegers(daysDiff(endHoliday, endWork));
        } else if (endYear == endYearHoliday && startYear < startYearHoliday) {
            workDaysWorked = getValueOrNegativeOneForIntegers(daysDiff(startWork, startHoliday));
            holidaysWorked = getValueOrNegativeOneForIntegers(daysDiff(startHoliday, endWork));
        } else if (endYear == endYearHoliday && startYear > startYearHoliday) {
            workDaysWorked = 0;
            holidaysWorked = getValueOrNegativeOneForIntegers(daysDiff(startWork, endWork));
        } else if (startYear == startYearHoliday && endYear == endYearHoliday) {
            if (startMonth > endMonthHoliday || startMonthHoliday > endMonth) {
                holidaysWorked = 0;
                workDaysWorked = getValueOrNegativeOneForIntegers(daysDiff(startWork, endWork));
            } else if (startMonth < startMonthHoliday && endMonth < endMonthHoliday) {
                workDaysWorked = Math.abs((monthToDays(startMonthHoliday) + startDayHoliday) - (monthToDays(startMonth) + startDay));
                holidaysWorked = Math.abs((monthToDays(endMonth) + endDay) - (monthToDays(startMonthHoliday) + startDayHoliday));
            } else if (startMonth > startMonthHoliday && endMonth > endMonthHoliday) {
                holidaysWorked = Math.abs((monthToDays(endMonthHoliday) + endDayHoliday) - (monthToDays(startMonth) + startDay));
                workDaysWorked = Math.abs((monthToDays(endMonth) + endDay) - (monthToDays(endMonthHoliday) + endDayHoliday));
            } else if (startMonth < startMonthHoliday && endMonth > endMonthHoliday) {
                holidaysWorked = Math.abs((monthToDays(startMonthHoliday) + startDayHoliday) - (monthToDays(startMonth) + startDay));
                workDaysWorked = Math.abs((monthToDays(startMonth) + startDay + monthToDays(endMonth) + endDay) - holidaysWorked);
            } else if (startMonth > startMonthHoliday && endMonth < endMonthHoliday) {
                holidaysWorked = (monthToDays(endMonth) + endDay) - (monthToDays(startMonth) + startDay);
                workDaysWorked = 0;
            } else if (startMonth == startMonthHoliday && endMonth < endMonthHoliday) {
                workDaysWorked = 0;
                holidaysWorked = Math.abs((monthToDays(endMonth) + endDay));
            } else if (startMonth == startMonthHoliday && endMonth > endMonthHoliday) {//todo here
                workDaysWorked = (monthToDays(endMonthHoliday) + endDayHoliday);
                holidaysWorked = Math.abs((monthToDays(endMonth) + endDay) - (monthToDays(endMonthHoliday) + endDayHoliday));
            } else if (startMonth > startMonthHoliday && endMonth == endMonthHoliday) {
                holidaysWorked = ((monthToDays(endMonth) + endDay) - monthToDays(startMonth) + startDay);
                workDaysWorked = 0;
            } else if (startMonth < startMonthHoliday && endMonth == endMonthHoliday) {
                workDaysWorked = Math.abs((monthToDays(startMonth) + startDay) - monthToDays(startMonthHoliday) + startDayHoliday);
                holidaysWorked = Math.abs(holidaysWorked - (monthToDays(endMonth) + endDay));
            } else if (startMonth == startMonthHoliday && endMonth == endMonthHoliday) {
                if (endDay < startDayHoliday || endDayHoliday < startDay) {
                    workDaysWorked = (endDay - startDay) + 1;
                    holidaysWorked = 0;
                } else if (startDay == startDayHoliday && endDay == endDayHoliday) {
                    workDaysWorked = 0;
                    holidaysWorked = (endDay - startDay) + 1;
                } else if (startDay < startDayHoliday && endDay < endDayHoliday) {
                    workDaysWorked = Math.abs((startDayHoliday - startDay)) + 1;
                    holidaysWorked = Math.abs((endDay - startDayHoliday)) + 1;
                } else if (startDay > startDayHoliday && endDay > endDayHoliday) {
                    workDaysWorked = Math.abs((endDay - endDayHoliday) + 1);
                    holidaysWorked = Math.abs((endDayHoliday - startDay) + 1);
                } else if (startDay < startDayHoliday && endDay > endDayHoliday) {
                    holidaysWorked = (endDayHoliday - startDayHoliday) + 1;
                    workDaysWorked = (endDay - startDay) - holidaysWorked;
                } else if (startDay > startDayHoliday && endDay < endDayHoliday) {
                    holidaysWorked = (endDay - startDay) + 1;
                    workDaysWorked = 0;
                } else if (startDay == startDayHoliday && endDay < endDayHoliday) {
                    workDaysWorked = 0;
                    holidaysWorked = Math.abs(endDay - startDay) +1;
                }else if (startDay == startDayHoliday && endDay > endDayHoliday) {
                    holidaysWorked = Math.abs(endDayHoliday - startDay) +1;
                    workDaysWorked = Math.abs(endDay - holidaysWorked);
                }else if (endDayHoliday == endDay && startDay > startDayHoliday) {
                    holidaysWorked = Math.abs(endDayHoliday - startDay) +1;
                    workDaysWorked = 0;
                }else if (endDayHoliday == endDay && startDay < startDayHoliday) {
                    workDaysWorked = Math.abs(startDayHoliday - startDay)+1;
                    holidaysWorked = Math.abs(endDayHoliday - startDayHoliday) +1;
                }
            }
        }
    }

    private void setWorkDaysWorked(String daysDiff) {
        workDaysWorked = getValueOrNegativeOneForIntegers(daysDiff);
    }

    private void setHolidaysWorked(String days) {
        holidaysWorked = getValueOrNegativeOneForIntegers(days);
    }

    public boolean checkJulianDate(String inDate) {
        String day, year;
        int _day;
        if (inDate.length() == 7) {
            day = inDate.charAt(4) + "" + inDate.charAt(5) + "" + inDate.charAt(6);
            year = inDate.charAt(0) + "" + inDate.charAt(1) + "" + inDate.charAt(2) + "" + inDate.charAt(3);
            _day = getValueOrNegativeOneForIntegers(day);
        } else {
            println("7 digits please");
            return false;
        }
        if (_day < 1 || _day > 365 && !getIsItALeapYear(getValueOrNegativeOneForIntegers(year)) || year.length() != 4) {
            println("Enter an integer that works please, Thank you!");
            return false;
        } else if (_day < 1 || _day > 366 && getIsItALeapYear(getValueOrNegativeOneForIntegers(year)) || year.length() != 4) {
            println("Enter an integer that works please, Thank you!");
            return false;
        }

        println("Your Julian date is good!");
        return true;
    }

    private boolean doesDayFitInMonth(int day, int month, int year) {
        isItALeapYear = getIsItALeapYear(year);

        switch (month) {
            case 1:
            case 3:
            case 5:
            case 7:
            case 8:
            case 10:
            case 12:
                if (day > 31) {
                    return false;
                }
                break;//All months with 31 days
            case 2://February
                if (isItALeapYear && day > 29) {
                    return false;
                } else if (!isItALeapYear && day > 28) {
                    return false;
                }
                break;
            case 4:
            case 6:
            case 9:
            case 11:
                if (day > 30) {
                    return false;
                }
                break;//All months with 30 days
        }
        return true;
    }

    public String gregorianDateToJulianDate(String inDate) {
        String day = inDate.charAt(6) + "" + inDate.charAt(7);
        String month = inDate.charAt(4) + "" + inDate.charAt(5);
        String year = inDate.charAt(0) + "" + inDate.charAt(1) + "" + inDate.charAt(2) + "" + inDate.charAt(3);
        int _day = getValueOrNegativeOneForIntegers(day);
        int _month = getValueOrNegativeOneForIntegers(month);

        int totalDays = _day + monthToDays(_month);
        String julianDate;
        if (totalDays < 100 && totalDays > 9) {
            julianDate = year + "0" + totalDays + "";
        } else if (totalDays < 10) {
            julianDate = year + "00" + totalDays + "";
        } else {
            julianDate = year + "" + totalDays + "";
        }
        return julianDate;
    }

    public String julianDateToGregorianDate(String inDate) {
        int month = 0;
        String year = inDate.charAt(0) + "" + inDate.charAt(1) + "" + inDate.charAt(2) + "" + inDate.charAt(3);
        String day = inDate.charAt(4) + "" + inDate.charAt(5) + "" + inDate.charAt(6);
        String gregorianDate;
        int _day = getValueOrNegativeOneForIntegers(day);
        isItALeapYear = getIsItALeapYear(getValueOrNegativeOneForIntegers(year));
        for (int i = 1; i <= 12; i++) {
            if (numberOfDaysInMonth(i) < _day) {
                _day -= numberOfDaysInMonth(i);
                month++;
            } else if (numberOfDaysInMonth(i) == _day) {
                month++;
                break;
            } else {
                month++;
                break;
            }
        }
        if (_day < 10 && month < 10) {
            gregorianDate = year + "0" + month + "0" + _day;
        } else if (_day < 10 && month >= 10) {
            gregorianDate = year + "" + month + "0" + _day;
        } else if (_day >= 10 && month < 10) {
            gregorianDate = year + "0" + month + "" + _day;
        } else {
            gregorianDate = year + "" + month + "" + _day;
        }
        return gregorianDate;
    }

    public String dayOfTheWeekFromGregorianDate(String inDate) {
        String day = inDate.charAt(6) + "" + inDate.charAt(7);//k - day
        String month = inDate.charAt(4) + "" + inDate.charAt(5);//month
        String year = inDate.charAt(2) + "" + inDate.charAt(3);//year
        int k = getValueOrNegativeOneForIntegers(day);
        int m = getValueOrNegativeOneForIntegers(month);
        int y = getValueOrNegativeOneForIntegers(year);
        int t[] = {0, 3, 2, 5, 0, 3, 5, 1, 4, 6, 2, 4};
        y -= (m < 3) ? 1 : 0;
        int w = ((y + y / 4 - y / 100 + y / 400 + t[m - 1] + k) % 7);
        return getDayOfTheWeek(w);
    }

    public String daysDiff(String date1, String date2) {
        date1 = gregorianDateToJulianDate(date1);
        date2 = gregorianDateToJulianDate(date2);
        String year1 = date1.charAt(0) + "" + date1.charAt(1) + "" + date1.charAt(2) + "" + date1.charAt(3);
        String day1 = date1.charAt(4) + "" + date1.charAt(5) + "" + date1.charAt(6);
        String year2 = date2.charAt(0) + "" + date2.charAt(1) + "" + date2.charAt(2) + "" + date2.charAt(3);
        String day2 = date2.charAt(4) + "" + date2.charAt(5) + "" + date2.charAt(6);

        int _year1 = getValueOrNegativeOneForIntegers(year1);
        int _day1 = getValueOrNegativeOneForIntegers(day1);
        int _year2 = getValueOrNegativeOneForIntegers(year2);
        int _day2 = getValueOrNegativeOneForIntegers(day2);
        int yearDifference = Math.abs(_year1 - _year2);
        int yearDifferenceInDays = (yearDifference / 4) + (365 * yearDifference);
        int dayDifferenceIs = yearDifferenceInDays + (Math.abs(_day1 - _day2));

        if (getIsItALeapYear(_year1)) {
            dayDifferenceIs++;
        }
        if (getIsItALeapYear(_year2)) {
            dayDifferenceIs++;
        }
        return dayDifferenceIs + "";
    }

    private boolean getIsItALeapYear(int year) {
        if (year % 4 == 0) {
            return true;
        }
        return false;
    }

    private String getDayOfTheWeek(int w) {
        switch (w) {

            case 0:
                return "Sunday";
            case 1:
                return "Monday";
            case 2:
                return "Tuesday";
            case 3:
                return "Wednesday";
            case 4:
                return "Thursday";
            case 5:
                return "Friday";
            case 6:
                return "Saturday";
        }
        return null;
    }

    private int monthToDays(int month) {
        int days = 0;
        for (int i = 1; i <= month - 1; i++) {
            days += numberOfDaysInMonth(i);
        }
        return days;
    }

    private int numberOfDaysInMonth(int month) {
        switch (month) {
            case 1:
            case 3:
            case 5:
            case 7:
            case 8:
            case 10:
            case 12:
                return 31;
            case 2://February
                if (isItALeapYear) {
                    return 29;
                }
                return 28;
            case 4:
            case 6:
            case 9:
            case 11:
                return 30;
        }
        return -1;
    }

    private void println(String text) {
        System.out.println(text);
    }


    private int getValueOrNegativeOneForIntegers(String input) {
        int result;
        try {
            result = Integer.valueOf(input);
        } catch (NumberFormatException exception) {
            result = -1;
        }
        return result;
    }
}
```

### Testing
```java
import java.util.Scanner;

public class Test {
    private static Scanner kb = new Scanner(System.in);

    public static void main(String[] args) {
        DateUtil test = new DateUtil();
        int userChoice;
        do {
            System.out.println("[0] to quit \n" +
                    "[1] To Check Your gregorian date\n" +
                    "[2] To check your julian date\n" +
                    "[3] To convert gregorian to Julian\n" +
                    "[4] To convert Julian to Gregorian\n" +
                    "[5] To find what day of the week your gregorian date occurred in\n" +
                    "[6] To find the difference between two dates in DAYS\n" +
                    "[7] To find the number of days worked in a normal work schedule && \n" +
                    "number of days worked during a holiday");
            userChoice = kb.nextInt();
            switch (userChoice) {
                case 0:
                    quit();
                    break;
                default:
                case 1:
                    System.out.println("Enter Gregorian Date Ex yyyymmdd");
                    if (test.checkGregorianDate(kb.next())) {
                        System.out.println("Your date is Valid good job");
                    } else {
                        System.out.println("Invalid Gregorian Date");
                    }
                    System.out.println("Process Complete...");
                    break;
                case 2:
                    System.out.println("Enter Julian Date Ex yyyyddd");
                    if (test.checkJulianDate(kb.next())) {
                        System.out.println("Your date is Valid good job");
                    } else {
                        System.out.println("Invalid Julian Date");
                    }
                    System.out.println("Process Complete...");
                    break;
                case 3:
                    System.out.println("Enter Gregorian Date Ex yyyymmdd");
                    String gregorianDate = kb.next();
                    if (test.checkGregorianDate(gregorianDate)) {
                        System.out.println("Your Julian date is: " + test.gregorianDateToJulianDate(gregorianDate));
                        System.out.println("Process Complete...");
                    } else {
                        System.out.println("Sorry can't process this since it's incorrect");
                        System.out.println("Process Complete...");
                    }
                    break;
                case 4:
                    System.out.println("Enter Julian Date Ex yyyyddd");
                    String julianDate = kb.next();
                    if (test.checkJulianDate(julianDate)) {
                        System.out.println("Your Gregorian date is: " + test.julianDateToGregorianDate(julianDate));
                        System.out.println("Process Complete...");

                    } else {
                        System.out.println("Sorry can't process this since it's incorrect");
                    }
                    break;
                case 5:
                    System.out.println("Enter Gregorian Date Ex yyyymmdd");
                    String gregorianDate2 = kb.next();
                    if (test.checkGregorianDate(gregorianDate2)) {
                        System.out.println("Your date is Valid, I will continue");
                        System.out.println("Your date tells me that that day of the week is: " + test.dayOfTheWeekFromGregorianDate(gregorianDate2));
                    } else {
                        System.out.println("Invalid Gregorian Date");
                    }
                    System.out.println("Process Complete...");
                    break;
                case 6:
                    String startDate, endDate;
                    do {
                        System.out.println("Enter start date(Gregorian): ");
                        startDate = kb.next();
                    } while (!test.checkGregorianDate(startDate));
                    do {
                        System.out.println("Enter end date(Gregorian): ");
                        endDate = kb.next();
                    } while (!test.checkGregorianDate(endDate));
                    System.out.println("The difference between the two dates is: " + test.daysDiff(startDate, endDate));
                    break;
                case 7:
                    String startDateWork, endDateWork, startDateHoliday, endDateHoliday;
                    do {
                        System.out.println("Enter start work date(Gregorian): ");
                        startDateWork = kb.next();
                    } while (!test.checkGregorianDate(startDateWork));
                    do {
                        System.out.println("Enter end work date(Gregorian): ");
                        endDateWork = kb.next();
                    } while (!test.checkGregorianDate(endDateWork));
                    do {
                        System.out.println("Enter start holiday date(Gregorian): ");
                        startDateHoliday = kb.next();
                    } while (!test.checkGregorianDate(startDateHoliday));
                    do {
                        System.out.println("Enter end holiday date(Gregorian): ");
                        endDateHoliday = kb.next();
                    } while (!test.checkGregorianDate(endDateHoliday));
                    test.daysTypeCount(startDateWork, endDateWork, startDateHoliday, endDateHoliday);
                    System.out.println("The days you worked on the holidays are: " + test.getHolidaysWorked());
                    System.out.println("The days you worked on your normal work days are: " + test.getWorkDaysWorked());
                    break;
            }
        } while (userChoice != 0);

    }

    private static void quit() {
        System.out.print("App is shutting down....");
    }
}
```