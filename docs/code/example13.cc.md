```C++
#include <iostream>
#include <string>
#include "freeling/morfo/util.h"
#include "freeling/morfo/tagset.h"

using namespace std;

int main (int argc, char *argv[]) {

  /// set locale to an UTF8 compatible locale
  freeling::util::init_locale(L"default");

  if (argc!=3) {
    wcerr<<L"Usage:  tagset tagset-file (tag2msd|msd2tag)  "<<endl;
    exit(1);
  }
  
  // get arguments
  wstring fname = freeling::util::string2wstring(argv[1]);
  wstring action = freeling::util::string2wstring(argv[2]);

  // create tagset handling module
  freeling::tagset tgs(fname);
 
  // convert tags to morphosyntactic descriptions
  if (action==L"tag2msd") {
    wstring form,lemma,tag;
    while (wcin >> form >> lemma >> tag) {
      wcout << form << L" " << lemma << L" " << tgs.get_msd_string(tag) << endl;
    }
  }
  // convert morphosyntactic descriptions to tags
  else if (action==L"msd2tag") {   
    wstring form,lemma,msd;
    while (wcin >> form >> lemma >> msd) {
      wcout << form << L" " << lemma << L" " << tgs.msd_to_tag(L"",msd) << endl;
    }
  }
  else 
    wcerr << L"Invalid action "<< action << endl;
  
}
```
