/**
 * Originally from https://github.com/jorgebucaran/getopts (35dfad8)
 *
 * Slightly modified to work with Bitburner.
 */

const EMPTYARR = []
const SHORTSPLIT = /$|[!-@[-`{-~][\s\S]*/g
const isArray = Array.isArray

const parseValue = function (any) {
  if (any === "") return ""
  if (any === "false") return false
  const maybe = +any
  return maybe * 0 === 0 ? maybe : any
}

const parseAlias = function (aliases) {
  let out = {},
    alias,
    prev,
    any

  for (let key in aliases) {
    any = aliases[key]
    alias = out[key] = isArray(any) ? any : [any]

    for (let i = 0; i < alias.length; i++) {
      prev = out[alias[i]] = [key]

      for (let k = 0; k < alias.length; k++) {
        if (i !== k) prev.push(alias[k])
      }
    }
  }

  return out
}

const parseDefault = function (aliases, defaults) {
  let out = {},
    alias,
    value

  for (let key in defaults) {
    alias = aliases[key]
    value = defaults[key]

    out[key] = value

    if (alias === undefined) {
      aliases[key] = EMPTYARR
    } else {
      for (let i = 0; i < alias.length; i++) {
        out[alias[i]] = value
      }
    }
  }

  return out
}

const parseOptions = function (aliases, options, value) {
  let out = {},
    key,
    alias

  if (options !== undefined) {
    for (let i = 0; i < options.length; i++) {
      key = options[i]
      alias = aliases[key]

      out[key] = value

      if (alias === undefined) {
        aliases[key] = EMPTYARR
      } else {
        for (let k = 0, end = alias.length; k < end; k++) {
          out[alias[k]] = value
        }
      }
    }
  }

  return out
}

const write = function (out, key, value, aliases, unknown) {
  let prev,
    alias = aliases[key],
    len = alias === undefined ? -1 : alias.length

  if (len >= 0 || unknown === undefined || unknown(key)) {
    prev = out[key]

    if (prev === undefined) {
      out[key] = value
    } else {
      if (isArray(prev)) {
        prev.push(value)
      } else {
        out[key] = [prev, value]
      }
    }

    for (let i = 0; i < len; i++) {
      out[alias[i]] = out[key]
    }
  }
}

export default function getopts(argv, opts) {
  let unknown = (opts = opts || {}).unknown,
    aliases = parseAlias(opts.alias),
    strings = parseOptions(aliases, opts.string, ""),
    values = parseDefault(aliases, opts.default),
    bools = parseOptions(aliases, opts.boolean, false),
    stopEarly = opts.stopEarly,
    _ = [],
    out = { _ },
    key,
    arg,
    end,
    match,
    value

  for (let i = 0, len = argv.length; i < len; i++) {
    arg = argv[i]

    if (arg[0] !== "-" || arg === "-") {
      if (stopEarly) {
        while (i < len) {
          _.push(argv[i++])
        }
      } else {
        _.push(arg)
      }
    } else if (arg === "--") {
      while (++i < len) {
        _.push(argv[i])
      }
    } else if (arg[1] === "-") {
      end = arg.indexOf("=", 2)
      if (arg[2] === "n" && arg[3] === "o" && arg[4] === "-") {
        key = arg.slice(5, end >= 0 ? end : undefined)
        value = false
      } else if (end >= 0) {
        key = arg.slice(2, end)
        value =
          bools[key] !== undefined ||
          (strings[key] === undefined
            ? parseValue(arg.slice(end + 1))
            : arg.slice(end + 1))
      } else {
        key = arg.slice(2)
        value =
          bools[key] !== undefined ||
          (len === i + 1 || argv[i + 1][0] === "-"
            ? strings[key] === undefined
              ? true
              : ""
            : strings[key] === undefined
            ? parseValue(argv[++i])
            : argv[++i])
      }
      write(out, key, value, aliases, unknown)
    } else {
      SHORTSPLIT.lastIndex = 2
      match = SHORTSPLIT.exec(arg)
      end = match.index
      value = match[0]

      for (let k = 1; k < end; k++) {
        write(
          out,
          (key = arg[k]),
          k + 1 < end
            ? strings[key] === undefined ||
                arg.substring(k + 1, (k = end)) + value
            : value === ""
            ? len === i + 1 || argv[i + 1][0] === "-"
              ? strings[key] === undefined || ""
              : bools[key] !== undefined ||
                (strings[key] === undefined ? parseValue(argv[++i]) : argv[++i])
            : bools[key] !== undefined ||
              (strings[key] === undefined ? parseValue(value) : value),
          aliases,
          unknown
        )
      }
    }
  }

  for (let key in values) if (out[key] === undefined) out[key] = values[key]
  for (let key in bools) if (out[key] === undefined) out[key] = false
  for (let key in strings) if (out[key] === undefined) out[key] = ""

  return out
}

////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////

'use script';

//	-----------	1	-----------
    //alert("Я JavaScript!");
//	-----------	2	-----------
    // let admin, name; // можно объявить две переменные через запятую
    // let OurPlanet, CurentUser;
    // name = "Джон";

    // admin = name;

    // alert( admin ); // "Джон"

//	-----------	3	-----------
    // let name = "Ilya";

    // alert( `hello ${1}` ); // ?

    // alert( `hello ${"name"}` ); // ?

    // alert( `hello ${name}` ); // ?

//	-----------	4	-----------
    // let name = prompt("Ваше имя?", "");
    // alert(`Привет ${name}`);
//	-----------	5	-----------
    // let a = 1, b = 1;

    // let c = --a; // ?
    // let d = b--; // ?
    // alert(c);
    // alert(d);
//	-----------	6	-----------
    // let a = 2;

    // let x = 1 + (a *= 2);
    // alert(x);
//	-----------	7	-----------
    // let a = prompt("Первое число?", 1);
    // let b = prompt("Второе число?", 2);

    // alert(+a + +b); // 12
//	-----------	8	-----------
    // 5 > 4 → true
    // "ананас" > "яблоко" → false
    // "2" > "12" → true
    // undefined == null → true
    // undefined === null → false
    // null == "\n0\n" → false
    // null === +"\n0\n" → false
//	-----------	9	-----------
//ddsdasd
//	-----------	10	-----------
// if ("0") {
//     alert( 'Привет' );
//   }
//	-----------	11	-----------
// let value = prompt('Какое "официальное" название JavaScript?', '');
// if (value == 'ECMAScript') {
//   alert('Верно!');
// } else {
//   alert('Не знаете? ECMAScript!');
// }
//	-----------	12	-----------
// let value = prompt('Введите число', 0);

// if (value > 0) {
//   alert( 1 );
// } else if (value < 0) {
//   alert( -1 );
// } else {
//   alert( 0 );
// }
//	-----------	13	-----------
// let result;

// if (a + b < 4) {
//   result = 'Мало';
// } else {
//   result = 'Много';
// }
// result = (a + b < 4) ? 'Мало' : 'Много';
//	-----------	14	-----------
// let message;

// if (login == 'Сотрудник') {
//   message = 'Привет';
// } else if (login == 'Директор') {
//   message = 'Здравствуйте';
// } else if (login == '') {
//   message = 'Нет логина';
// } else {
//   message = '';
// }
// message =   (login == 'Сотрудник') ?'Привет':
//             (login == 'Сотрудник') ?'Здравствуйте':
//             (login == '') ?'Нет логина':'';
//	-----------	15	-----------
// alert( null || 2 || undefined );
//	-----------	16	-----------
// alert( 1 && null && 2 );
// alert( alert(1) && alert(2) );
// alert( null || 2 && 3 || 4 );
// if (age >= 14 && age <= 90);
// if (!(age >= 14 && age <= 90))
//	-----------	17	-----------
// if (-1 || 0) alert( 'first' );
// if (-1 && 0) alert( 'second' );
// if (null || -1 && 1) alert( 'third' );
//	-----------	18	-----------
    // let user = prompt("Who are u?","");
    // if (user == "Me"){
    //     let paswrd = prompt("Password pls", "");
    //     if (paswrd=="111"){
    //         alert("Welcame")

    //     }
    //     else if (paswrd == "" || user == null){
    //         alert("wrong");
    //     }
    //     else {
    //         alert("wrong paswrd");
    //     }}
    // else if (user == "" || user == null){
    //     alert("wrong");
    // }
    // else {
    //     alert("idk u");
    // }
    // const name = prompt('Who is it?', '');
    // if(name === 'Admin' || name && alert('I don\'t know you') || !name && alert('Rejected')) {
    // const password = prompt('Password', '');
    // if(password === 'I am the man'|| password && alert('Wrong password') 
    // || !password && alert('Rejected') ) alert('Hello!')
    // }
//	-----------	19	-----------
// let login, password;

// do {
// login = prompt("Login");

// if (login == null || login == "") {
//     alert ("Canceled!");
//     alert ("Please, try again!");
//     } 
// else if (login != "admin") {
//     alert ("I don't know you!");
//     alert ("Please, try again!");
//     }} 
// while (login != "admin");  
//     console.log(`Login: ${login}`);

// if (login == "admin") {
// do {
// password = prompt("Password");
// if (password == null || password == "") {
// alert("Canceled");
// break;
// } else if (password != "Boss") {
// alert ("Wrong password!");
// alert ("try again!");
// }
// } while (password != "Boss");
// }

// console.log(`Password: ${password}`);
//	-----------	20	-----------
    // let login, password;

    // function WritePasvrd(){
    //     switch(password){
    //     case "1":  
    //     alert ("Welcome");   
    //     break;
    //     case "2":  
    //     alert ("Welcome");   
    //     break;

    //     default:
    //         alert ("Wrong pasvord");
    //         alert ("Please, try again!");
    //     break;
    // }
    // }

    // login = prompt("Login");
    // switch(login){
    //     case "1":  
    //         password = prompt("Password");
    //         alert(password==2 ||password==1 ? "We;come" : "Wrong pasvord")
    //     break;
    //     case "2":   
    //         password = prompt("Password");
    //         alert(password==2 ||password==1 ? "We;come" : "Wrong pasvord")
    //     break;
    //     default:
    //         alert ("I don't know you!");
    //         alert ("Please, try again!");
    //     break;
    // }

//	-----------	21	-----------
let user1 = {
    login:'111',
    password:'111'
};
let user2 = {
    login:'222',
    password:'222'
};
login = prompt("Login");
if(login==user1.login ||login==user2.login){
    password= prompt("Password");
    if(password==user1.password ||password==user2.password){
        alert('Welcome');
        
    }
    else{
        alert('Wrong pasword');
    }
}
else{
    alert('I do not know u');
}


//	-----------	22	-----------
//	-----------	23	-----------
//	-----------	24	-----------
//	-----------	25	-----------
//	-----------	26	-----------
//	-----------	27	-----------
//	-----------	28	-----------
//	-----------	29	-----------
//	-----------	30	-----------
