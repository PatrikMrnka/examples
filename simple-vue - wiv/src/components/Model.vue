<template>
    <section>
        <input type="file" id="file-input" />
        <p id="properties-text">
            ID: 
            {{ entityData }}
        </p>
        <div id="model" />
    </section>
</template>

<script>
import { IfcViewerAPI } from 'web-ifc-viewer';
import {IFCWALL,
  IFCWALLSTANDARDCASE,
  IFCSLAB,
  IFCWINDOW,
  IFCCURTAINWALL,
  IFCFURNISHINGELEMENT,
  IFCDOOR,
    IFCMEMBER,
    IFCPLATE
} from 'web-ifc';

export default {
    name: 'Model',
    props: ['token', 'projectId', 'discipline'],
    data() {
        return {
            entityData: '',
        }
    },
    methods: {
    },
    mounted() {

      const container = document.getElementById('model');
      const viewer = new IfcViewerAPI({ container });
      viewer.axes.setAxes();
      viewer.grid.setGrid();
      viewer.IFC.setWasmPath('../IFCjs/');

      const input = document.getElementById("file-input");

      input.addEventListener("change",

          async (changed) => {
            const file = event.target.files[0];
            const url = URL.createObjectURL(file);

            // Export to glTF and JSON
            const result = await viewer.GLTF.exportIfcFileAsGltf({
              ifcFileUrl: url,
              splitByFloors: true,
              categories: {
                walls: [IFCWALL, IFCWALLSTANDARDCASE],
                slabs: [IFCSLAB],
                windows: [IFCWINDOW],
                curtainwalls: [IFCMEMBER, IFCPLATE, IFCCURTAINWALL],
                furniture: [IFCFURNISHINGELEMENT],
                doors: [IFCDOOR]
              },
              getProperties: true
            });

            // Download result
            const link = document.createElement('a');
            document.body.appendChild(link);

            for(const categoryName in result.gltf) {
              const category = result.gltf[categoryName];
              for(const levelName in category) {
                const file = category[levelName].file;
                if(file) {
                  link.download = `${file.name}_${categoryName}_${levelName}.gltf`;
                  link.href = URL.createObjectURL(file);
                  link.click();
                }
              }
            }

            for(let jsonFile of result.json) {
              link.download = `${jsonFile.name}.json`;
              link.href = URL.createObjectURL(jsonFile);
              link.click();
            }

            link.remove();
          },

          false
      );
    },
}
</script>

<style>
#model {
    position: absolute;
    left: 0%;
    top: 0%;
    width: 100% !important;
    height: 100% !important;
}

#file-input {
    position: absolute;
    left: 0%;
    top: 0%;
    z-index: 100;
}

#properties-text {
    position: absolute;
    left: 0%;
    bottom: 0%;
    z-index: 100;
}
</style>
