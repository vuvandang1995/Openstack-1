#!/bin/bash
# -*- encoding: utf-8; py-indent-offset: 4 -*-
# +------------------------------------------------------------------+
# |             ____ _               _        __  __ _  __           |
# |            / ___| |__   ___  ___| | __   |  \/  | |/ /           |
# |           | |   | '_ \ / _ \/ __| |/ /   | |\/| | ' /            |
# |           | |___| | | |  __/ (__|   <    | |  | | . \            |
# |            \____|_| |_|\___|\___|_|\_\___|_|  |_|_|\_\           |
# |                                                                  |
# | Copyright Mathias Kettner 2014             mk@mathias-kettner.de |
# +------------------------------------------------------------------+



factory_settings['hh3c_entity_temp_default_levels'] = {
    'levels': (61, 66),
}

def _hh3c_entity_temp_get_name_for_sensor(nr, ent_physical_descr, module):
    if ent_physical_descr in ['SENSOR']:
        description = 'module'
    elif ent_physical_descr in ['Temperature Sensor on Board']:
        description = 'module board'
    else:
        return None
    # return '{} {}: {}, warn: {}, critical: {}'.format(description, int(module), temp, warn, critical)
    return '{} {} {}'.format(nr, description, int(module))

def inventory_hh3c_entity_temp(info):
    # Debug: lets see how the data we get looks like
    # import pprint ; pprint.pprint(info)

    module = 1.0
    for nr, elem in enumerate(info):
        ent_physical_descr, temp = elem
        temp = int(temp)
        name = _hh3c_entity_temp_get_name_for_sensor(nr, ent_physical_descr, module)
        if temp != 65535 and name:
            # print(name)
            yield name, {}

            module += 0.5
            ## Occurs two times per module. Different temperatures … See above.

def check_hh3c_entity_temp(item, params, info):
    for nr, elem in enumerate(info):
        if unicode(item).startswith(unicode(nr) + ' '):
            ent_physical_descr, temp = elem
            degree_celsius = float(elem[1])
            return check_temperature(degree_celsius, params, 'hh3c_entity_temp_{}'.format(item))

    return (3, "Invalid information in snmp data")

check_info['hh3c_entity_temp'] = {
    'check_function'          : check_hh3c_entity_temp,
    'inventory_function'      : inventory_hh3c_entity_temp,
    "service_description"     : "Temperature %s",
    'snmp_scan_function'      : lambda oid: 'Comware Platform Software' in oid('.1.3.6.1.2.1.1.1.0') and \
        oid('.1.3.6.1.4.1.25506.2.6.1.1.1.1.12.1') != None,
    'snmp_info'               : ( '.1.3.6.1',
        [
            '2.1.47.1.1.1.1.2',
            '4.1.25506.2.6.1.1.1.1.12',
            # '4.1.25506.2.6.1.1.1.1.13',
            # '4.1.25506.2.6.1.1.1.1.17',
        ] ),
    'has_perfdata'            : True,
    'default_levels_variable' : 'hh3c_entity_temp_default_levels',
    'group'                   : 'temperature',
    "includes"                : [ 'temperature.include' ],
}
