#include <stdio.h>
#include <stdlib.h>
#include <string.h>
 int check_mod(char *mod) {
    if (mod[0] >= 'A' && mod[0] <= 'Z' && (mod[1] >= '0' && mod[1] <= '9') && (mod[2] >= '0' && mod[2] <= '9')) {
        int number = (mod[1] - '0') * 10 + (mod[2] - '0');
        if (number >= 0 && number <= 99) {
            return 0;
        } else
        {
            return 1;
        }
    } else {
        return 1;
    }

}

int check_typ(char *typ) {
    const char *valid_inputs[] = {"R1", "U1", "A1", "R2", "U2", "A2", "R4", "U4", "A4"};
    int num_valid_inputs = sizeof(valid_inputs) / sizeof(valid_inputs[0]);

    for (int i = 0; i < num_valid_inputs; i++) {
        if (strcmp(typ, valid_inputs[i]) == 0) {
            return 0;
        }
    }
    return 1;
}

int check_cas(long cas) {
    int hour = cas / 100;
    int minute = cas % 100;

    if (hour >= 0 && hour <= 24 && minute >= 0 && minute <= 59) {
        return 0;
    }
    return 1;
}

int check_datum(long long datum) {
    long long year = datum / 10000;
    long long month = (datum / 100) % 100;
    long long day = datum % 100;

    if (year >= 0 && month >= 1 && month <= 12 && day >= 1 && day <= 31) {
        // Simple check for the number of days in each month (ignoring leap years)
        if ((month == 4 || month == 6 || month == 9 || month == 11) && day > 30) {
            return 1; // Invalid day for April, June, September, November
        } else if (month == 2 && day > 29) {
            return 1; // Invalid day for February
        } else if (month == 2 && day == 29 && (year % 4 != 0 || (year % 100 == 0 && year % 400 != 0))) {
            return 1; // Invalid day for non-leap year February
        }
        return 0;
    }
    return 1;
}

void s_function(char ***mer_mod, char ***typ_mer, float **hodnota, long **cas, long long **datum, int counter) {
    if (counter < 0) {
        printf("Polia nie su vytvorene\n");
        return;
    }

    char mer_mod_scan[4];
    char typ_scan[3];
    scanf("%s %s", mer_mod_scan, typ_scan);

    int counter_help = 0;
    float hodnota_help[30];
    long cas_help[30];
    long long datum_help[30];

    for (int i = 0; i < counter; i++) {
        if (strcmp((*mer_mod)[i], mer_mod_scan) == 0 && strcmp((*typ_mer)[i], typ_scan) == 0) {
            counter_help++;
            hodnota_help[counter_help] = (*hodnota)[i];
            cas_help[counter_help] = (*cas)[i];
            datum_help[counter_help] = (*datum)[i];
        }
    }

    long long help_datum;
    long help_cas;
    float help_hodnota;
    int min;

    for (int j = 0; j < counter_help - 1; j++) {
        min = j;
        for (int k = j + 1; k < counter_help; k++) {
            if (datum_help[k] * 10000 + cas_help[k] < datum_help[min] * 10000 + cas_help[min]) {
                min = k;
            }
        }

        help_datum = datum_help[j];
        help_cas = cas_help[j];
        help_hodnota = hodnota_help[j];

        datum_help[j] = datum_help[min];
        cas_help[j] = cas_help[min];
        hodnota_help[j] = hodnota_help[min];

        datum_help[min] = help_datum;
        cas_help[min] = help_cas;
        hodnota_help[min] = help_hodnota;
    }

    // Create and write sorted measurements to output file
    FILE *new_file = fopen("vystup_S.txt", "w");
    if(counter_help == 0)
    {
        printf("Pre dany vstup neexistuju zaznamy\n");
        fclose(new_file);
        return ;
    }

    for (int j = 1; j <= counter_help; j++) {
        fprintf(new_file, "%lld%04ld     %0.7f\n", datum_help[j], cas_help[j], hodnota_help[j]);
    }

    fclose(new_file);
    printf("Pre dany vrtup je vytvoreny txt subor\n");
}

int n_function(FILE **file, long long **id, char ***mer_mod, char ***typ_mer, float **hodnota, long **cas, long long **datum, int opened, int counter)
{
    if(opened < 0)
    {
        printf("Neotvoreny subor\n");
        return -1;
    }


    if(counter > 0)
    {
        for (int i = 0; i < counter; i++)
        {
            free((*mer_mod)[i]);
            free((*typ_mer)[i]);

        }
        free(*id);
        free(*mer_mod);
        free(*typ_mer);
        free(*hodnota);
        free(*cas);
        free(*datum);

        *id = NULL;
        *mer_mod = NULL;
        *typ_mer = NULL;
        *hodnota = NULL;
        *cas = NULL;
        *datum = NULL;
    }

    int count;
    int result = 1;

    while ((count = fgetc(*file)) != EOF)
    {
        if(count == '\n')
        {
            result++;
        }
    }
    result = result/7;
    fseek(*file, 0, SEEK_SET);

    *id = (long long *) malloc(result * sizeof(long long));
    *mer_mod = (char **) malloc(result * sizeof(char *));
    for (int i = 0; i < result; i++)
    {
        (*mer_mod) [i] = (char *) malloc(3 * sizeof(char));
    }
    *typ_mer = (char **) malloc(result * sizeof(char *));
    for (int i = 0; i < result; i++)
    {
        (*typ_mer) [i] = (char *) malloc(3 * sizeof(char));
    }
    *hodnota = (float *) malloc(result * sizeof(float));
    *cas = (long *) malloc(result * sizeof(long));
    *datum = (long long *) malloc (result * sizeof(long long));

    for (int i = 0; i < result; i++)
    {
        fscanf(*file, "%lld", &(*id)[i]);
        fscanf(*file, "%s", (*mer_mod)[i]);
        fscanf(*file, "%s", (*typ_mer)[i]);
        fscanf(*file, "%f", &(*hodnota)[i]);
        fscanf(*file, "%ld", &(*cas)[i]);
        fscanf(*file, "%lld", &(*datum)[i]);

    }
    fseek(*file, 0, SEEK_SET);
    return result;
}

int v_function(FILE **file, long long **id, char ***mer_mod, char ***typ_mer, float **hodnota, long **cas, long long **data, int counter)
{

    *file = fopen("dataloger.txt", "r");
    if(*file == NULL)
    {
        printf("Neotvoreny subor\n");
        return -1;
    }


    if(counter < 0)
    {
        char c[12];
        int i = 1;
        while (fscanf(*file, "%s", c) != EOF)
        {
            printf("ID cislo mer. osoby: %s\n", c);
            i++;

            fscanf(*file, "%s", c);
            printf("Mer. modul: %s\n", c);
            i++;

            fscanf(*file, "%s", c);
            printf("Typ mer. veliciny: %s\n", c);
            i++;

            fscanf(*file, "%s", c);
            printf("Hodnota: %s\n", c);
            i++;

            fscanf(*file, "%s", c);
            printf("Cas merania: %s\n", c);
            i++;

            fscanf(*file, "%s", c);
            printf("Datum: %s\n\n", c);
            i++;

        }
        fseek(*file, 0, SEEK_SET);
    } else
    {
        for (int i=0; i < counter; i++)
        {
            printf("ID cislo mer. osoby: %lld\n", (*id)[i]);
            printf("Mer. modul: %s\n", (*mer_mod)[i]);
            printf("Typ mer. veliciny: %s\n", (*typ_mer)[i]);
            printf("Hodnota: %lf\n", (*hodnota)[i]);
            printf("Cas merania: %04ld\n", (*cas)[i]);
            printf("Datum: %lld\n\n", (*data)[i]);
        }
    }
    return 1;

}

void o_function(FILE **file, int opened)
{
    if(opened < 0 )
    {
        printf("Neotvoreny subor\n");
        return;
    }
    char mer_mod_scan[4];
    char typ_scan[3];
    scanf("%s %s", mer_mod_scan, typ_scan);

        int counter_help = 0;
        char mod_help[4];
        char typ_help[3];
        float hodnota_help[30];
        long cas_help[30];
        long long datum_help[30];

        int i = 0;
        char c[12];

        while (fscanf(*file, "%s", c) != EOF) {
            i++;
            i++;
            fscanf(*file, "%s\n", mod_help);
            i++;
            fscanf(*file, "%s\n", typ_help);
            i++;

            if (strcmp(mer_mod_scan, mod_help) == 0 && strcmp(typ_scan, typ_help) == 0) {
                counter_help++;
                fscanf(*file, "%f", &hodnota_help[counter_help]);
                i++;
                fscanf(*file, "%04ld", &cas_help[counter_help]);
                i++;
                fscanf(*file, "%lld", &datum_help[counter_help]);
                i++;
            } else
            {
                fscanf(*file, "%s", c);
                i++;
                fscanf(*file, "%s", c);
                i++;
                fscanf(*file, "%s", c);
                i++;
            }
        }
            fseek(*file, 0, SEEK_SET);

        long long help_datum = 0;
        long help_cas = 0;
        float help_hodnota = 0;
        int min;

        for ( int j = 0; j < counter_help-1; j++)
        {
            min = j;
            for(int k = j+1; k < counter_help; k++) {
                if (datum_help[k] * 10000 + cas_help[k] < datum_help[min] * 10000 + cas_help[min]) {
                    min = k;
                }
            }

        help_datum = datum_help[j];
        help_cas = cas_help[j];
        help_hodnota = hodnota_help[j];

        datum_help[j] = datum_help[min];
        cas_help[j] = cas_help[min];
        hodnota_help[j] = hodnota_help[min];

        datum_help[min] = help_datum;
        cas_help[min] = help_cas;
        hodnota_help[min] = help_hodnota;
        }


        for(int j=1; j <= counter_help; j ++)
        {
            printf("%3s %3s     %lld     %04ld     %f\n", mer_mod_scan, typ_scan, datum_help[j], cas_help[j], hodnota_help[j]);

        }


}

void c_function(FILE **file, int opened)
{
    if(opened < 0)
    {
        printf("Neotvoreny subor\n");
        return;
    }

    long long id_d;
    long long bad_id[60];
    int count_bad_id = 0;

    char mod_d[4];
    char bad_mod[60];
    int count_bad_mod = 0;

    char typ_p[4];
    char bad_typ[60];
    int count_bad_typ = 0;

    long cas_s;
    long bad_cas[60];
    int count_bad_cas = 0;

    long long datum_m;
    long long bad_datum[60];
    int count_bad_datum = 0;

    char c[12];
    int i = 1;
    while (fscanf(*file, "%lld", &id_d) != EOF)
    {
        if(id_d%11 != 0)
        {
            bad_id[count_bad_id] = id_d;
            count_bad_id++;
        }
//        printf("ID cislo mer. osoby: %s\n", c);
        i++;

        fscanf(*file, "%s", mod_d);
        if(check_mod(mod_d) == 1)
        {
            strcpy(&bad_mod[count_bad_mod], mod_d);
            count_bad_mod ++;
        }
//        printf("Mer. modul: %s\n", c);
        i++;

        fscanf(*file, "%s", typ_p);
        if(check_typ(typ_p) == 1)
        {
            strcpy(&bad_typ[count_bad_typ], typ_p);
            count_bad_typ ++;
        }

//        printf("Typ mer. veliciny: %s\n", c);
        i++;

        fscanf(*file, "%s", c);
//        printf("Hodnota: %s\n", c);
        i++;

        fscanf(*file, "%ld", &cas_s);
        if(check_cas(cas_s) == 1)
        {
            bad_cas[count_bad_cas] = cas_s;
            count_bad_cas++;
        }
//        printf("Cas merania: %s\n", c);
        i++;

        fscanf(*file, "%lld", &datum_m);
        if(check_datum(datum_m) == 1)
        {
            bad_datum[count_bad_datum] = datum_m;
            count_bad_datum++;
        }
//        printf("Datum: %s\n\n", c);
        i++;

    }
    fseek(*file, 0, SEEK_SET);

    if (count_bad_id > 0) {
        printf("Nekorektne zadany vstup: ");
        for (int j = 0; j < count_bad_id; j++) {
            if(j == count_bad_id -1)
            {
                printf("%lld\n", bad_id[j]);
            } else {
                printf("%lld ", bad_id[j]);
            }
        }
    }

    // Print bad_mod array if not empty
    if (count_bad_mod > 0) {
        printf("Nekorektne zadany vstup: ");
        for (int j = 0; j < count_bad_mod; j++) {
            if(j == count_bad_mod -1)
            {
                printf("%s\n", &bad_mod[j]);
            } else {
                printf("%s ", &bad_mod[j]);
            }
        }
    }

    // Print bad_typ array if not empty
    if (count_bad_typ > 0) {
        printf("Nekorektne zadany vstup: ");
        for (int j = 0; j < count_bad_typ; j++) {
            if(j == count_bad_typ -1)
            {
                printf("%s\n", &bad_typ[j]);
            } else {
                printf("%s ", &bad_typ[j]);
            }
        }
    }

    // Print bad_cas array if not empty
    if (count_bad_cas > 0) {
        printf("Nekorektne zadany vstup: ");
        for (int j = 0; j < count_bad_cas; j++) {
            if(j == count_bad_cas -1)
            {
                printf("%ld\n", bad_cas[j]);
            } else {
                printf("%ld ", bad_cas[j]);
            }
        }
    }

    // Print bad_datum array if not empty
    if (count_bad_datum > 0) {
        printf("Nekorektne zadany vstup: ");
        for (int j = 0; j < count_bad_datum; j++) {
            if(j == count_bad_datum -1)
            {
                printf("%lld\n", bad_datum[j]);
            } else
            {
                printf("%lld ", bad_datum[j]);
            }
        }
    }


}

int z_function(long long **id, char ***mer_mod, char ***typ_mer, float **hodnota, long **cas, long long **datum, int counter) {
    if (counter < 0) {
        printf("Polia nie su vytvorene\n");
        return counter;
    }

    long long id_scan;

    scanf("%lld", &id_scan);

    int deleted = 0;
    int new_counter = counter; // Initialize new_counter

    for (int i = 0; i < counter; i++) {
        if ((*id)[i] != id_scan) {
            (*id)[i - deleted] = (*id)[i]; // Update the id
            (*mer_mod)[i - deleted] = (*mer_mod)[i];
            (*typ_mer)[i - deleted] = (*typ_mer)[i];
            (*hodnota)[i - deleted] = (*hodnota)[i];
            (*cas)[i - deleted] = (*cas)[i];
            (*datum)[i - deleted] = (*datum)[i];
        } else {
            deleted++;

            free((*mer_mod)[i]);
            free((*typ_mer)[i]);
        }
    }

    // Adjust the new counter after deletion
    new_counter -= deleted;

    // Reallocate memory for the arrays
    *id = (long long *)realloc(*id, new_counter * sizeof(long long));
    // Similar reallocations for other arrays
    *mer_mod = (char **)realloc(*mer_mod, new_counter * sizeof(char *));
    *typ_mer = (char **)realloc(*typ_mer, new_counter * sizeof(char *));
    *hodnota = (float *)realloc(*hodnota, new_counter * sizeof(float));
    *cas = (long *)realloc(*cas, new_counter * sizeof(long));
    *datum = (long long *)realloc(*datum, new_counter * sizeof(long long));

    // Check for reallocation failures and handle them

    printf("Vymazalo sa: %d zaznamov !\n", deleted);

    return new_counter;
}

void k_function(FILE *file, long long *id, char **mer_mod, char **typ_mer, float *hodnota, long *cas, long long *datum, int opened, int counter)
 {
    if(counter > 0)
    {
        for(int i = 0; i < counter; i++)
        {
            free(mer_mod[i]);
            free(typ_mer[i]);
        }
        free(id);
        free(mer_mod);
        free(typ_mer);
        free(hodnota);
        free(cas);
        free(datum);
    }
    if(opened > 0)
    {
        fclose(file);
    }
}

void r_function(long *cas, int counter)
{
    if(counter < 0)
    {
        printf("Polia nie su vytvorene\n");
        return;
    }

    int time[24][60];

    for (int i = 0; i < 24; i++)
    {
        for (int j = 0; j < 60; j++)
        {
            time[i][j] = 0;
        }
    }

    for (int i = 0; i < counter; i++)
    {
        long hours = cas[i] / 100;
        long minutes = cas[i] % 100;

        if (time[hours][minutes] != 1)  // check and sign occupied minutes
        {
            time[hours][minutes] = 1;
            time[hours][59]++;
        }
    }

    for (int i = 0; i < 24; i++)
    {
        if (time[i][59] > 0)
        {
            printf("%02d:", i);
            for (int j = 0; j < 60; j++)
            {
                if (time[i][j] == 1)
                {
                    if (time[i][59] > 1)
                    {
                        printf("%02d, ", j);
                    } else {
                        printf("%02d\n", j);
                    }
                    time[i][59]--;
                }
            }
        }
    }
}

int main ()
{
    FILE  *file;
    long long *id = NULL;
    char **mer_mod = NULL;
    char **typ_mer = NULL;
    float *hodnota = NULL;
    long *cas = NULL;
    long long *datum = NULL;

    int opened = -1;
    int counter = -1;
    char change;
    printf("enter the operation:\n");
    scanf("%c", &change);
    while (change != 'k') { // DONE
        switch (change) {
            case 'v': // DONE
                opened = v_function(&file, &id, &mer_mod, &typ_mer, &hodnota, &cas, &datum, counter);
                break;
            case 'o': // DONE
                o_function(&file, opened);
                break;
            case 'n': // DONE
                counter = n_function(&file, &id, &mer_mod, &typ_mer, &hodnota, &cas, &datum, opened, counter);
                break;
            case 'c': // DONE
                c_function(&file, opened);
                break;
            case 's': // DONE
                s_function(&mer_mod, &typ_mer, &hodnota, &cas, &datum, counter);
                break;
            case 'h':
                break;
            case 'r': // DONE
                r_function(cas, counter);
                break;
            case 'z': // DONE
                counter = z_function(&id, &mer_mod, &typ_mer, &hodnota, &cas, &datum, counter);
                break;
        }
        scanf(" %c", &change);

    }
    if(change=='k')
    {
        k_function(file, id, mer_mod, typ_mer, hodnota, cas, datum, opened, counter);
    }

    return 0;
}