#####一些常用方法
######String
Remove the exclamations:

	#include <algorithm>
	#include <iterator>
	
	std::string result;
	std::remove_copy(delStr.begin(), delStr.end(), std::back_inserter(result), '!');

Alternatively, if you want to print the string, you don't need the result variable:

	#include <iostream>
	
	std::remove_copy(delStr.begin(), delStr.end(),
	                 std::ostream_iterator<char>(std::cout), '!');

Replace slashes with dashes:

	std::replace(repStr.begin(), repStr.end(), '/', '-');