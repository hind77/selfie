// === [ Source code representation ] ==========================================

// --- [ Characters ] ----------------------------------------------------------

newline        = /* the Unicode code point U+000A */ .
unicode_char   = /* an arbitrary Unicode code point except newline */ .
unicode_letter = /* a Unicode code point classified as "Letter" */ .
unicode_digit  = /* a Unicode code point classified as "Decimal Digit" */ .
ascii_char     = /* an arbitrary ASCII character except newline */ .

// --- [ Letters and digits ] --------------------------------------------------

letter        = unicode_letter | "_" .
binary_digit  = "0" … "1" .
octal_digit   = "0" … "7" .
decimal_digit = "0" … "9" .
hex_digit     = "0" … "9" | "A" … "F" | "a" … "f" .

// === [ Lexical elements ] ====================================================

// --- [ Comments ] ------------------------------------------------------------

// A line comment acts like a newline.
line_comment = ";" { unicode_char } newline .

// --- [ Identifiers ] ---------------------------------------------------------

identifier = letter { letter | unicode_digit } .

// --- [ Integer literals ] ----------------------------------------------------

// TODO(u): Add support for floating point literals.

int_lit     = [ "+" | "-" ] binary_lit | octal_lit | decimal_lit | hex_lit .
binary_lit  = "0b" binary_digit { binary_digit } .
octal_lit   = "0" { octal_digit } .
decimal_lit = ( "1" … "9" ) { decimal_digit } .
hex_lit     = "0" ( "x" | "X" ) hex_digit { hex_digit } .

// --- [ Character literals ] --------------------------------------------------

// A character literal is expressed as one or more characters enclosed in single
// quotes. Within the quotes, any character may appear except single quote and
// newline. A single quoted character represents the ASCII value of the
// character itself, while multi-character sequences beginning with a backslash
// encode values in various formats.
//
// Several backslash escapes allow arbitrary values to be encoded as ASCII text.
// There are two ways to represent the integer value as a numeric constant: \x
// followed by exactly two hexadecimal digits, and a plain backslash \ followed
// by exactly three octal digits. Although these representations all result in
// an integer, they have different valid ranges. Octal escapes must represent a
// value between 0 and 255 inclusive.
//
// After a backslash, certain single-character escapes represent special values:
//
//    \a   U+0007 alert or bell
//    \b   U+0008 backspace
//    \f   U+000C form feed
//    \n   U+000A line feed or newline
//    \r   U+000D carriage return
//    \t   U+0009 horizontal tab
//    \v   U+000b vertical tab
//    \\   U+005c backslash
//    \'   U+0027 single quote  (valid escape only within character literals)
//    \"   U+0022 double quote  (valid escape only within string literals)
//
// All other sequences starting with a backslash are illegal inside character
// literals.
char_lit         = "'" ( ascii_value | byte_value ) "'" .
ascii_value      = ascii_char | escaped_char .
byte_value       = octal_byte_value | hex_byte_value .
octal_byte_value = `\` octal_digit octal_digit octal_digit .
hex_byte_value   = `\` "x" hex_digit hex_digit .
escaped_char     = `\` ( "a" | "b" | "f" | "n" | "r" | "t" | "v" | `\` | "'" | `"` ) .

// --- [ String literals ] -----------------------------------------------------

string_lit             = raw_string_lit | interpreted_string_lit .
raw_string_lit         = "`" { ascii_char | newline } "`" .
interpreted_string_lit = `"` { ascii_value | byte_value } `"` .

// === [ Source code structure ] ===============================================

Program = { Line } .
Line    = [ LabelDecl ] [ Instruction | Directive ] [ line_comment ] .

// --- [ Label declaration ] ---------------------------------------------------

LabelDecl = Label ":" .

// Local labels starts with a ".".
Label = [ "." ] identifier .

// === [ Instructions ] ========================================================

// TODO(u): Add support for expressions.

// The definitions of Operator and Register are architecture specific.
Instruction = Operator [ Operand { "," Operand } ] .
Operand     = Register | Label | Immediate .

// The precise definition of an Operator is architecture specific.
Operator = identifier .

// The precise definition of a Register is architecture specific.
Register = [ "$" ] identifier .

// === [ Directives ] ==========================================================

// TODO(u): Write directive specifications.
//    section ".text"
//    align 0x80
Directive = .

// == [ Immediate values ] =====================================================

Immediate = int_lit | char_lit | string_lit | raw_string_lit .
